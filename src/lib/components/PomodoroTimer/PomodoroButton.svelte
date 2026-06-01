<script lang="ts">
    import type { HTMLButtonAttributes } from "svelte/elements";
    import { cn } from "$lib/utils";

    type PomodoroAction = "stop" | "pause" | "play";

    interface Props extends HTMLButtonAttributes {
        action: PomodoroAction;
        class?: string;
    }

    const { action, class: className, ...rest }: Props = $props();
</script>

<button
    type="button"
    class={cn(
        "flex size-16 items-center justify-center rounded-full bg-neutral-100 shadow-lg cursor-pointer transition-all hover:bg-neutral-200 active:scale-95",
        className,
    )}
    {...rest}
>
    {#if action === "stop"}
        <span class="size-5 rounded-md bg-accent-100"></span>
    {:else if action === "pause"}
        <span class="flex gap-1.5">
            <span class="h-6 w-2 rounded-sm bg-accent-200"></span>
            <span class="h-6 w-2 rounded-sm bg-accent-200"></span>
        </span>
    {:else}
        <svg viewBox="0 0 24 24" class="size-7 translate-x-0.5 fill-accent-200" aria-hidden="true">
            <path d="M8 5v14l11-7z" />
        </svg>
    {/if}
</button>
