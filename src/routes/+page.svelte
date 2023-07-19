<script lang="ts">
    import { onMount } from "svelte";
    let chat: any;
    let socket: any;
    let messages: Array<object> = []
    const API_URL: string = 'http://localhost:3000'
    onMount(() => {
        if (chat) {
            chat.scroll({ top: chat.scrollHeight })
        }
        socket = io(API_URL);

        socket.on('chat message', (msg: string) => {
            messages.push({
                body: msg
            })
            messages = messages
        })
    })

    function autoResize(e: any) {
        e.target.style.height = 'auto';
        e.target.style.height = e.target.scrollHeight + 'px'; // Set the height to the scrollHeight of the textarea
    }

    function handleKeyDown(e: any) {
        if (e.keyCode === 13 && !e.shiftKey) {
            e.preventDefault()
            socket.emit('chat message', e.target.value)
            e.target.value = ''
            chat.scroll({ top: chat.scrollHeight })
        }
    }
</script>

<div class="main-wrapper">
    <div class="container">
        <h1 class="title-xl">My ChatRoom</h1>
        <p class="subtitle">3 personnes connecté(e)s</p>
        <div bind:this={chat} class="chat-wrapper">
            {#each messages as message}
            <p class="right">
                {message.body}
            </p>
            {/each}
        </div>
        <div class="input-wrapper">
            <textarea on:input={autoResize} on:keydown={handleKeyDown} placeholder="Votre message" rows="1" cols="0" maxlength="500"></textarea>
            <img src="send.svg" alt="">
        </div>
    </div>
</div>

<style>
    :root {
        /* COLORS */
        --primary: #1D2C6B;
        --bg-gray: #F4F6F9;
        --dark: #333333;
        --white: #FFFFFF;

        /* SPACINGS */
        --spacing-top: 4.85rem;
        --default-letter-spacing: -0.06rem;

        /* GENERAL */
        --opacity-fading: .7;
        --font-size-xl: 3rem;
        --font-size-md: 1.25rem;
        --font-size-sm: 1rem;
    }

    :global(*) {
        letter-spacing: var(--default-letter-spacing);
        font-family: 'Roboto', sans-serif;
        color: var(--dark);
        margin: 0;
        padding: 0;
        box-sizing: border-box;
        border-radius: 0.625rem;
    }

    :global(body) {
        background: var(--bg-gray);
    }

    .container {
        max-width: 1200px;
        padding: 0 1.5rem;
        margin: auto;
    }

    .main-wrapper {
        margin-top: var(--spacing-top);
    }

    .title-xl {
        font-size: var(--font-size-xl);
        font-weight: 700;
        margin-bottom: .41rem;
    }

    .subtitle {
        font-size: var(--font-size-md);
        font-weight: 500;
        letter-spacing: -0.025rem;
        opacity: var(--opacity-fading);
    }

    .chat-wrapper {
        margin-top: 5.06rem;
        display: flex;
        flex-direction: column;
        max-height: 50vh;
        overflow-y: scroll;
        padding-right: 1rem;
    }

    .chat-wrapper p {
        margin-bottom: 1.3rem;
        /* padding: .88rem 0; */
        padding: .88rem 1.7rem;
        background: var(--white);
        max-width: 500px;
        font-weight: 400;
        letter-spacing: -0.02rem;
        font-size: var(--font-size-sm);
    }

    .right {
        margin-left: auto;
    }

    .input-wrapper {
        margin-top: 2rem;
        display: flex;
    }

    .input-wrapper textarea {
        resize: none;
        background-color: var(--white);
        border-radius: 0.625rem;
        padding: 1.02rem 1.81rem;
        font-size: var(--font-size-sm);
        letter-spacing: -0.02rem;
        font-weight: 400;
        flex: 1;
        gap: 1.29rem;
        outline: none;
        border: 0;
    }

    .input-wrapper textarea::placeholder {
        opacity: var(--opacity-fading);
    }

    .input-wrapper img {
        padding-left: 1.29rem;
        cursor: pointer;
    }

    .input-wrapper img.disabled {
        opacity: var(--opacity-fading);
    }
</style>