<script lang="ts">
    import { onMount } from "svelte";
    import { browser } from "$app/environment";
    import { page } from "$app/state";
    import { goto } from "$app/navigation";
    import { defaultNavPage } from "$lib/subnav";

    let errorCode = 500;
    let errorMessage = "Произошла непредвиденная ошибка";
    let showDetails = false;
    let redirecting = false;

    $: if (browser && page.error) {
        if (page.error.message === "Not Found") {
            errorCode = 404;
            errorMessage = "Страница не найдена";
        } else {
            errorCode = page.status ?? 500;
            errorMessage = page.error.message || "Произошла непредвиденная ошибка";
        }
    }

    onMount(() => {
        if (page.error?.message === "Not Found") {
            redirecting = true;
            if (page.url.pathname.startsWith("/settings")) {
                goto(defaultNavPage("settings"), { replaceState: true });
            } else if (page.url.pathname.startsWith("/about")) {
                goto(defaultNavPage("about"), { replaceState: true });
            } else {
                goto("/", { replaceState: true });
            }
        }
    });
</script>

{#if !redirecting}
<div class="error-page">
    <div class="error-container">
        <div class="error-code">{errorCode}</div>
        <h1 class="error-title">
            {#if errorCode === 404}
                Страница не найдена
            {:else if errorCode === 401}
                Необходима авторизация
            {:else if errorCode === 403}
                Доступ запрещён
            {:else if errorCode === 429}
                Слишком много запросов
            {:else if errorCode >= 500}
                Ошибка сервера
            {:else}
                Что-то пошло не так
            {/if}
        </h1>
        <p class="error-message">{errorMessage}</p>

        <div class="error-actions">
            <a href="/" class="error-button primary">На главную</a>
            <button
                class="error-button secondary"
                on:click={() => {
                    if (browser) window.location.reload();
                }}
            >
                Перезагрузить
            </button>
        </div>

        {#if errorCode >= 500}
            <button
                class="details-toggle"
                on:click={() => showDetails = !showDetails}
            >
                {showDetails ? "Скрыть" : "Показать"} подробности
            </button>
            {#if showDetails && page.error}
                <pre class="error-details">{page.error.message}</pre>
            {/if}
        {/if}
    </div>
</div>
{/if}

<style>
    .error-page {
        display: flex;
        align-items: center;
        justify-content: center;
        width: 100%;
        height: 100%;
        padding: 24px;
        background: var(--primary, #fff);
        color: var(--secondary, #000);
    }

    .error-container {
        display: flex;
        flex-direction: column;
        align-items: center;
        text-align: center;
        gap: 12px;
        max-width: 420px;
    }

    .error-code {
        font-size: 72px;
        font-weight: 800;
        line-height: 1;
        opacity: 0.15;
        letter-spacing: -2px;
    }

    .error-title {
        font-size: 22px;
        font-weight: 700;
        margin: 0;
    }

    .error-message {
        font-size: 14.5px;
        color: var(--gray, #757575);
        line-height: 1.6;
        margin: 0;
    }

    .error-actions {
        display: flex;
        gap: 10px;
        margin-top: 12px;
    }

    .error-button {
        padding: 10px 24px;
        border-radius: 14px;
        font-size: 14px;
        font-weight: 600;
        cursor: pointer;
        border: none;
        text-decoration: none;
        transition: opacity 0.15s;
    }

    .error-button:hover {
        opacity: 0.85;
    }

    .error-button.primary {
        background: var(--secondary, #000);
        color: var(--primary, #fff);
    }

    .error-button.secondary {
        background: var(--button, #f4f4f4);
        color: var(--button-text, #282828);
    }

    .details-toggle {
        background: none;
        border: none;
        color: var(--gray, #757575);
        font-size: 13px;
        cursor: pointer;
        margin-top: 8px;
        text-decoration: underline;
        text-underline-offset: 3px;
    }

    .error-details {
        background: var(--button, #f4f4f4);
        border-radius: 12px;
        padding: 14px;
        font-size: 12px;
        font-family: monospace;
        max-width: 100%;
        overflow-x: auto;
        text-align: left;
        white-space: pre-wrap;
        word-break: break-word;
    }
</style>
