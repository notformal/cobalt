<script lang="ts">
    import { onMount, onDestroy, tick } from "svelte";
    import { killDialog } from "$lib/state/dialogs";

    import DialogBackdropClose from "$components/dialog/DialogBackdropClose.svelte";

    export let id: string;
    export let dismissable = true;

    let dialogParent: HTMLDialogElement;

    let open = false;
    let closing = false;

    export const close = () => {
        if (dialogParent) {
            closing = true;
            open = false;

            setTimeout(() => {
                if (dialogParent) {
                    dialogParent.close();
                    killDialog();
                }
            }, 150);
        }
    };

    function handleKeydown(e: KeyboardEvent) {
        if (e.key === "Escape" && dismissable) {
            e.preventDefault();
            close();
            return;
        }

        if (e.key !== "Tab" || !dialogParent) return;

        const focusable = dialogParent.querySelectorAll<HTMLElement>(
            'button:not([disabled]), [href], input:not([disabled]), select:not([disabled]), textarea:not([disabled]), [tabindex]:not([tabindex="-1"])'
        );
        if (focusable.length === 0) return;

        const first = focusable[0];
        const last = focusable[focusable.length - 1];

        if (!dialogParent.contains(document.activeElement)) {
            e.preventDefault();
            first.focus();
            return;
        }

        if (e.shiftKey) {
            if (document.activeElement === first) {
                e.preventDefault();
                last.focus();
            }
        } else {
            if (document.activeElement === last) {
                e.preventDefault();
                first.focus();
            }
        }
    }

    $: if (dialogParent) {
        dialogParent.showModal();
        tick().then(() => {
            open = true;
            const firstFocusable = dialogParent?.querySelector<HTMLElement>(
                'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
            );
            firstFocusable?.focus();
        });
    }

    onMount(() => {
        document.addEventListener("keydown", handleKeydown);
    });

    onDestroy(() => {
        if (typeof document !== "undefined") {
            document.removeEventListener("keydown", handleKeydown);
        }
    });
</script>

<dialog
    id="dialog-{id}"
    bind:this={dialogParent}
    class:closing
    class:open
    aria-modal="true"
    role="dialog"
>
    <slot></slot>
    <DialogBackdropClose closeFunc={dismissable ? close : () => {}} />
</dialog>
