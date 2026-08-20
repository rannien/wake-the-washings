<script setup lang="ts">
import { ref, computed, onMounted, onUnmounted } from "vue";

// ── Helpers ───────────────────────────────────────────────────────────────────
function pad(n: number): string {
    return String(n).padStart(2, "0");
}
function toHM(totalMinutes: number): string {
    return `${Math.floor(totalMinutes / 60)}:${pad(totalMinutes % 60)}`;
}
function fmt(d: Date): string {
    return `${pad(d.getHours())}:${pad(d.getMinutes())}`;
}
function fmtFull(d: Date): string {
    return `${pad(d.getHours())}:${pad(d.getMinutes())}:${pad(d.getSeconds())}`;
}

// ── State ─────────────────────────────────────────────────────────────────────
const duration = ref<string>("90");
const finishTime = ref<string>("07:00");
const now = ref<Date>(new Date());
const isDark = ref<boolean>(false);
const durInput = ref<HTMLInputElement | null>(null);

let ticker: ReturnType<typeof setInterval>;
let darkMq: MediaQueryList;

onMounted(() => {
    ticker = setInterval(() => {
        now.value = new Date();
    }, 1000);

    darkMq = window.matchMedia("(prefers-color-scheme: dark)");
    isDark.value = darkMq.matches;
    darkMq.addEventListener("change", (e) => {
        isDark.value = e.matches;
    });

    durInput.value?.focus();
});

onUnmounted(() => {
    clearInterval(ticker);
    darkMq?.removeEventListener("change", () => {});
});

// ── Computed ──────────────────────────────────────────────────────────────────
const currentTimeDisplay = computed(() => fmtFull(now.value));

const result = computed(() => {
    const dur = parseInt(duration.value, 10);
    const fin = finishTime.value;
    if (!fin || isNaN(dur) || dur <= 0) return null;

    const [fh, fm] = fin.split(":").map(Number);
    let target = new Date(now.value);
    target.setHours(fh, fm, 0, 0);

    let isNextDay = false;
    if (
        target <= now.value ||
        target.getTime() - now.value.getTime() < 60_000
    ) {
        target.setDate(target.getDate() + 1);
        isNextDay = true;
    }

    const startMs = target.getTime() - dur * 60_000;
    const delayMin = Math.floor((startMs - now.value.getTime()) / 60_000);

    if (delayMin < 0) {
        return {
            valid: false as const,
            isNextDay,
            startTime: fmt(new Date(startMs)),
        };
    }

    const steps = Math.round(delayMin / 30);
    const roundedDelay = steps * 30;
    const expectedFinish = new Date(
        now.value.getTime() + roundedDelay * 60_000 + dur * 60_000,
    );

    return {
        valid: true as const,
        isNextDay,
        startTime: fmt(new Date(startMs)),
        delayDisplay: toHM(roundedDelay),
        expectedFinish: fmt(expectedFinish),
    };
});
</script>

<template>
    <!-- dark class here drives all dark: variants -->
    <div :class="{ dark: isDark }">
        <div
            class="min-h-screen bg-gray-50 dark:bg-gray-950 transition-colors duration-300 flex flex-col items-center justify-start px-4 py-8 sm:py-12"
        >
            <div class="w-full max-w-md">
                <!-- Header -->
                <div class="flex items-center justify-between mb-6">
                    <div class="flex items-center gap-3">
                        <div
                            class="w-11 h-11 rounded-xl bg-blue-100 dark:bg-blue-950 flex items-center justify-center flex-shrink-0"
                        >
                            <svg
                                xmlns="http://www.w3.org/2000/svg"
                                class="w-6 h-6 text-blue-500"
                                viewBox="0 0 24 24"
                                fill="none"
                                stroke="currentColor"
                                stroke-width="1.5"
                                stroke-linecap="round"
                                stroke-linejoin="round"
                            >
                                <rect
                                    x="2"
                                    y="2"
                                    width="20"
                                    height="20"
                                    rx="2"
                                />
                                <circle cx="12" cy="13" r="4" />
                                <path d="M5 6h1M8 6h1" />
                            </svg>
                        </div>
                        <div>
                            <h1
                                class="text-xl font-semibold text-gray-900 dark:text-white leading-tight"
                            >
                                Washing Machine Timer
                            </h1>
                            <p class="text-sm text-gray-500 dark:text-gray-400">
                                Delay start calculator
                            </p>
                        </div>
                    </div>

                    <!-- Dark mode toggle -->
                    <button
                        @click="isDark = !isDark"
                        class="w-9 h-9 rounded-xl flex items-center justify-center transition-colors bg-gray-200 dark:bg-gray-800 text-gray-600 dark:text-gray-300 hover:bg-gray-300 dark:hover:bg-gray-700"
                        :title="
                            isDark
                                ? 'Switch to light mode'
                                : 'Switch to dark mode'
                        "
                    >
                        <svg
                            v-if="isDark"
                            xmlns="http://www.w3.org/2000/svg"
                            width="18"
                            height="18"
                            viewBox="0 0 24 24"
                            fill="none"
                            stroke="currentColor"
                            stroke-width="2"
                            stroke-linecap="round"
                            stroke-linejoin="round"
                        >
                            <circle cx="12" cy="12" r="4" />
                            <path
                                d="M12 2v2M12 20v2M4.93 4.93l1.41 1.41M17.66 17.66l1.41 1.41M2 12h2M20 12h2M6.34 17.66l-1.41 1.41M19.07 4.93l-1.41 1.41"
                            />
                        </svg>
                        <svg
                            v-else
                            xmlns="http://www.w3.org/2000/svg"
                            width="18"
                            height="18"
                            viewBox="0 0 24 24"
                            fill="none"
                            stroke="currentColor"
                            stroke-width="2"
                            stroke-linecap="round"
                            stroke-linejoin="round"
                        >
                            <path
                                d="M21 12.79A9 9 0 1 1 11.21 3 7 7 0 0 0 21 12.79z"
                            />
                        </svg>
                    </button>
                </div>

                <!-- Input Card -->
                <div
                    class="bg-white dark:bg-gray-900 border border-gray-200 dark:border-gray-800 rounded-2xl p-5 mb-3 shadow-sm"
                >
                    <p
                        class="text-xs font-semibold text-gray-400 dark:text-gray-500 uppercase tracking-widest mb-4"
                    >
                        Settings
                    </p>
                    <div class="grid grid-cols-1 sm:grid-cols-2 gap-3">
                        <div>
                            <label
                                for="dur"
                                class="block text-xs font-medium text-gray-500 dark:text-gray-400 uppercase tracking-wide mb-1.5"
                            >
                                Program duration (min)
                            </label>
                            <input
                                id="dur"
                                ref="durInput"
                                type="number"
                                v-model="duration"
                                min="1"
                                max="999"
                                placeholder="e.g. 90"
                                class="w-full rounded-lg px-4 py-3 text-lg font-mono font-medium border transition-all focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-transparent bg-white dark:bg-gray-800 border-gray-200 dark:border-gray-700 text-gray-900 dark:text-white placeholder-gray-400 dark:placeholder-gray-600"
                            />
                        </div>
                        <div>
                            <label
                                for="fin"
                                class="block text-xs font-medium text-gray-500 dark:text-gray-400 uppercase tracking-wide mb-1.5"
                            >
                                Finish by
                            </label>
                            <input
                                id="fin"
                                type="time"
                                v-model="finishTime"
                                class="w-full rounded-lg px-4 py-3 text-lg font-mono font-medium border transition-all focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-transparent bg-white dark:bg-gray-800 border-gray-200 dark:border-gray-700 text-gray-900 dark:text-white"
                            />
                        </div>
                    </div>
                </div>

                <!-- Results Card -->
                <div
                    class="bg-white dark:bg-gray-900 border border-gray-200 dark:border-gray-800 rounded-2xl p-5 shadow-sm"
                >
                    <div class="flex items-center justify-between mb-4">
                        <p
                            class="text-xs font-semibold text-gray-400 dark:text-gray-500 uppercase tracking-widest"
                        >
                            Result
                        </p>
                        <span
                            v-if="result?.isNextDay"
                            class="text-xs font-medium px-2.5 py-1 rounded-full bg-amber-100 dark:bg-amber-950 text-amber-700 dark:text-amber-400"
                        >
                            tomorrow
                        </span>
                    </div>

                    <!-- Current time + Start time -->
                    <div class="grid grid-cols-2 gap-3 mb-3">
                        <div class="bg-gray-50 dark:bg-gray-800 rounded-xl p-4">
                            <div class="flex items-center gap-1.5 mb-1.5">
                                <span
                                    class="w-2 h-2 rounded-full bg-green-400 flex-shrink-0"
                                    style="
                                        animation: blink 1.4s ease-in-out
                                            infinite;
                                    "
                                ></span>
                                <span
                                    class="text-xs font-medium text-gray-400 dark:text-gray-500 uppercase tracking-wide"
                                    >Current time</span
                                >
                            </div>
                            <span
                                class="font-mono text-xl font-medium tabular-nums text-gray-900 dark:text-white"
                            >
                                {{ currentTimeDisplay }}
                            </span>
                        </div>
                        <div class="bg-blue-50 dark:bg-blue-950 rounded-xl p-4">
                            <div class="flex items-center gap-1.5 mb-1.5">
                                <svg
                                    xmlns="http://www.w3.org/2000/svg"
                                    class="w-3.5 h-3.5 text-blue-500 dark:text-blue-400"
                                    viewBox="0 0 24 24"
                                    fill="none"
                                    stroke="currentColor"
                                    stroke-width="2"
                                    stroke-linecap="round"
                                    stroke-linejoin="round"
                                >
                                    <polygon points="5 3 19 12 5 21 5 3" />
                                </svg>
                                <span
                                    class="text-xs font-medium text-blue-500 dark:text-blue-400 uppercase tracking-wide"
                                    >Start time</span
                                >
                            </div>
                            <span
                                class="font-mono text-xl font-medium tabular-nums text-blue-700 dark:text-blue-300"
                            >
                                {{ result?.startTime ?? "--:--" }}
                            </span>
                        </div>
                    </div>

                    <div
                        class="border-t border-gray-100 dark:border-gray-800 my-3"
                    />

                    <!-- Valid result -->
                    <template v-if="result?.valid">
                        <div class="bg-blue-600 rounded-xl p-4 mb-3">
                            <div class="flex items-center gap-1.5 mb-2">
                                <svg
                                    xmlns="http://www.w3.org/2000/svg"
                                    class="w-3.5 h-3.5 text-blue-200"
                                    viewBox="0 0 24 24"
                                    fill="none"
                                    stroke="currentColor"
                                    stroke-width="2"
                                    stroke-linecap="round"
                                    stroke-linejoin="round"
                                >
                                    <circle cx="12" cy="12" r="10" />
                                    <polyline points="12 6 12 12 16 14" />
                                </svg>
                                <span
                                    class="text-xs font-medium text-blue-200 uppercase tracking-wide"
                                    >Set this delay</span
                                >
                            </div>
                            <span
                                class="font-mono text-5xl font-semibold text-white tabular-nums tracking-tight"
                            >
                                {{ result.delayDisplay }}
                            </span>
                        </div>
                        <div class="bg-gray-50 dark:bg-gray-800 rounded-xl p-4">
                            <div class="flex items-center gap-1.5 mb-1.5">
                                <svg
                                    xmlns="http://www.w3.org/2000/svg"
                                    class="w-3.5 h-3.5 text-gray-400 dark:text-gray-500"
                                    viewBox="0 0 24 24"
                                    fill="none"
                                    stroke="currentColor"
                                    stroke-width="2"
                                    stroke-linecap="round"
                                    stroke-linejoin="round"
                                >
                                    <path
                                        d="M12 22c5.523 0 10-4.477 10-10S17.523 2 12 2 2 6.477 2 12s4.477 10 10 10z"
                                    />
                                    <path d="m9 12 2 2 4-4" />
                                </svg>
                                <span
                                    class="text-xs font-medium text-gray-400 dark:text-gray-500 uppercase tracking-wide"
                                    >Expected finish</span
                                >
                            </div>
                            <span
                                class="font-mono text-xl font-medium tabular-nums text-gray-700 dark:text-gray-200"
                            >
                                {{ result.expectedFinish }}
                            </span>
                        </div>
                    </template>

                    <!-- Invalid -->
                    <template v-else-if="result && !result.valid">
                        <div
                            class="bg-red-50 dark:bg-red-950 border border-red-100 dark:border-red-900 rounded-xl p-4 text-sm text-red-600 dark:text-red-400"
                        >
                            <div class="flex items-start gap-2">
                                <svg
                                    xmlns="http://www.w3.org/2000/svg"
                                    class="w-4 h-4 mt-0.5 flex-shrink-0"
                                    viewBox="0 0 24 24"
                                    fill="none"
                                    stroke="currentColor"
                                    stroke-width="2"
                                    stroke-linecap="round"
                                    stroke-linejoin="round"
                                >
                                    <path
                                        d="m21.73 18-8-14a2 2 0 0 0-3.48 0l-8 14A2 2 0 0 0 4 21h16a2 2 0 0 0 1.73-3Z"
                                    />
                                    <line x1="12" y1="9" x2="12" y2="13" />
                                    <line x1="12" y1="17" x2="12.01" y2="17" />
                                </svg>
                                <span
                                    >The program won't fit before the selected
                                    finish time. Choose a later finish time or a
                                    shorter program.</span
                                >
                            </div>
                        </div>
                    </template>

                    <!-- Empty -->
                    <template v-else>
                        <div
                            class="py-6 text-center text-sm text-gray-400 dark:text-gray-600"
                        >
                            Enter program duration and desired finish time.
                        </div>
                    </template>
                </div>

                <p
                    class="text-center text-xs text-gray-400 dark:text-gray-600 mt-5"
                >
                    If the selected time has already passed, the app calculates
                    for tomorrow.
                </p>
            </div>
        </div>
    </div>
</template>

<style scoped>
@keyframes blink {
    0%,
    100% {
        opacity: 1;
    }
    50% {
        opacity: 0.3;
    }
}
</style>
