<template lang="pug">
h1 Welcome, gyupp-site! 과제 테스트입니다.
p 아래 버튼을 누르면 서버의 다른 Git 프로젝트에서 strong git pull --ff-only 를 실행합니다.

.row
    button#pullBtn(type="button" :disabled="isPulling" @click="handlePull") 최신 코드 Pull

pre#result {{ resultText }}
</template>

<script setup lang="ts">
import { ref } from "vue";

const isPulling = ref(false);
const resultText = ref("대기 중...");

const handlePull = async () => {
    isPulling.value = true;
    resultText.value = "git pull 실행 시작...\n\n";

    try {
        const res = await fetch("https://api.gyupp.com/api/pull/stream", {
            method: "POST",
            headers: {
                "Content-Type": "application/json"
            }
        });

        if (!res.ok) {
            const errorText = await res.text();
            resultText.value += `요청 실패 (${res.status})\n${errorText}`;
            return;
        }

        if (!res.body) {
            resultText.value += "스트리밍 본문이 없습니다.";
            return;
        }

        const reader = res.body.getReader();
        const decoder = new TextDecoder();

        while (true) {
            const { done, value } = await reader.read();
            if (done) {
                break;
            }

            resultText.value += decoder.decode(value, { stream: true });
        }

        resultText.value += "\n스트리밍 종료";
    } catch (err) {
        const message = err instanceof Error ? err.message : String(err);
        resultText.value = `요청 실패: ${message}`;
    } finally {
        isPulling.value = false;
    }
};
</script>

<style scoped lang="less">
body {
    font-family: sans-serif;
    margin: 2rem;
    line-height: 1.5;
}

.row {
    display: flex;
    gap: 0.5rem;
    flex-wrap: wrap;
    align-items: center;
    margin-top: 1rem;
}

input {
    min-width: 18rem;
    padding: 0.5rem;
}

button {
    padding: 0.5rem 0.9rem;
    cursor: pointer;
}

pre {
    margin-top: 1rem;
    padding: 0.75rem;
    background: #f4f4f4;
    white-space: pre-wrap;
    border-radius: 6px;
}
</style>
