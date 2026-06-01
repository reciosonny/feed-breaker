<script lang="ts">
    import toast from "svelte-french-toast";
    import { cn } from "$lib/utils";
    import PomodoroIcon from "./PomodoroIcon.svelte";
    import PomodoroButton from "./PomodoroButton.svelte";
    import "./pomodoro-timer.scss";

    type Status = "idle" | "running" | "paused";

    interface Props {
        title?: string;
        durationMinutes?: number;
        rounds?: number;
        onComplete?: () => void;
        class?: string;
    }

    const {
        title = "Simple task",
        durationMinutes = 25,
        rounds = 3,
        onComplete,
        class: className,
    }: Props = $props();

    // SVG ring geometry (viewBox 0 0 100 100, radius 45)
    const RADIUS = 45;
    const CIRCUMFERENCE = 2 * Math.PI * RADIUS;

    let status = $state<Status>("idle");
    let remainingSeconds = $state(durationMinutes * 60);

    const totalSeconds = $derived(durationMinutes * 60);
    const progress = $derived(totalSeconds > 0 ? remainingSeconds / totalSeconds : 0);
    // Elapsed (cyan) sweeps clockwise from the top; remaining (red) is what's left.
    // The white knob rides the boundary between them.
    const elapsedDashOffset = $derived(CIRCUMFERENCE * progress);
    const angle = $derived((1 - progress) * 360);

    const pad = (n: number) => String(n).padStart(2, "0");
    const label = $derived(`${pad(Math.floor(remainingSeconds / 60))}:${pad(remainingSeconds % 60)}`);

    const isActive = $derived(status !== "idle");
    const roundDots = $derived(Array.from({ length: rounds }, (_, i) => i));

    // Component-scoped interval handle — mirrors the runInterval pattern in App.svelte.
    let intervalId: number;

    const tick = () => {
        remainingSeconds--;

        if (remainingSeconds <= 0) {
            clearInterval(intervalId);
            remainingSeconds = totalSeconds;
            status = "idle";
            toast.success("Session complete!");
            onComplete?.();
        }
    };

    const run = () => {
        status = "running";
        clearInterval(intervalId);
        intervalId = setInterval(tick, 1000);
    };

    const pause = () => {
        clearInterval(intervalId);
        status = "paused";
    };

    const stop = () => {
        clearInterval(intervalId);
        remainingSeconds = totalSeconds;
        status = "idle";
    };

    $effect(() => () => clearInterval(intervalId));
</script>

<div class={cn("flex flex-col items-center gap-6 pb-12", className)}>
    <h2 class="heading-sm text-neutral-500 text-center">{title}</h2>

    <div class="pt-ring size-72" data-animate={status === "running"} style:--angle={`${angle}deg`}>
        <!-- Ring -->
        <svg class="size-full" viewBox="0 0 100 100">
            {#if isActive}
                <!-- Remaining (red) underlay + elapsed (cyan) sweep on top -->
                <circle class="stroke-accent-100" cx="50" cy="50" r={RADIUS} fill="none" stroke-width="6" />
                <circle
                    class="pt-arc stroke-accent-200"
                    cx="50"
                    cy="50"
                    r={RADIUS}
                    fill="none"
                    stroke-width="6"
                    stroke-linecap="round"
                    stroke-dasharray={CIRCUMFERENCE}
                    stroke-dashoffset={elapsedDashOffset}
                    transform="rotate(-90 50 50)"
                />
            {:else}
                <!-- Idle: calm full-cyan ring -->
                <circle class="stroke-accent-200" cx="50" cy="50" r={RADIUS} fill="none" stroke-width="6" />
            {/if}
        </svg>

        <!-- Boundary marker: idle time badge, or the orbiting knob while active -->
        <div class="pt-orbit">
            <div class="pt-badge">
                {#if isActive}
                    <span class="block size-7 rounded-full border-2 border-neutral-300 bg-white shadow-md"></span>
                {:else}
                    <span class="flex size-12 items-center justify-center rounded-full bg-accent-100 text-white body-md">
                        {label}
                    </span>
                {/if}
            </div>
        </div>

        <!-- Center -->
        <div class="absolute inset-0 flex flex-col items-center justify-center gap-4">
            {#if isActive}
                <span class="heading-xl font-bold text-neutral-700 leading-none tabular-nums">{label}</span>
                <div class="flex gap-2">
                    {#each roundDots as i (i)}
                        <PomodoroIcon class={cn("size-6", i === 0 ? "text-accent-100" : "text-neutral-400")} />
                    {/each}
                </div>
            {:else}
                <button
                    type="button"
                    aria-label="Start timer"
                    onclick={run}
                    class="cursor-pointer transition-opacity hover:opacity-80"
                >
                    <svg viewBox="0 0 24 24" class="size-16 fill-primary-200" aria-hidden="true">
                        <path d="M8 5v14l11-7z" />
                    </svg>
                </button>
            {/if}
        </div>

        <!-- Controls straddling the ring's bottom edge (active only) -->
        {#if isActive}
            <div class="absolute bottom-0 left-1/2 flex -translate-x-1/2 translate-y-1/2 gap-4">
                <PomodoroButton action="stop" aria-label="Stop timer" onclick={stop} />
                {#if status === "running"}
                    <PomodoroButton action="pause" aria-label="Pause timer" onclick={pause} />
                {:else}
                    <PomodoroButton action="play" aria-label="Resume timer" onclick={run} />
                {/if}
            </div>
        {/if}
    </div>
</div>
