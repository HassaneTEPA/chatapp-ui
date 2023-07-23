<script lang="ts">
    import { onMount } from "svelte";
    let chat: any;
    let socket: any;
    let messages: Array<object> = []
    const API_URL: string = 'http://localhost:3000'
    let user_id: string | undefined = undefined
    let users_connected: number = 0
    let username_input = ''

    let username = ''

    onMount(() => {
        if (chat) {
            chat.scroll({ top: chat.scrollHeight })
        }
        socket = io(API_URL);

        socket.on('user_has_logged_in', (nb_users: number) => {
            users_connected = nb_users
        })

        socket.on('chat message', (data: any) => {
            messages.push({
                body: data[0],
                isMine: false,
                username: data[1]
            })
            messages = messages
        })

        socket.on('init user', (guest_id: string) => {
            user_id = guest_id
        })
    })

    function autoResize(e: any) {
        e.target.style.height = 'auto';
        e.target.style.height = e.target.scrollHeight + 'px'; // Set the height to the scrollHeight of the textarea
    }

    function handleKeyDown(e: any) {
        if (e.keyCode === 13 && !e.shiftKey) {
            e.preventDefault()
            socket.emit('chat message', [e.target.value, user_id])
            messages.push({body: e.target.value, isMine: true})
            messages = messages
            e.target.value = ''
            setTimeout(() => {
                chat.scrollTo(0, chat.scrollHeight)
            }, 50)
        }
    }

    function handleSubmit(e: any) {
        e.preventDefault()
        socket.emit('change_username', username_input)
        username = username_input
    }
</script>

<div class="main-wrapper">
    <div class="container">
        <h1 class="title-xl">My ChatRoom</h1>
        <p class="subtitle mb-lg">{users_connected} personnes connecté(e)s</p>
        <div bind:this={chat} class="chat-wrapper">
            {#each messages as message}
            <p class={message.isMine ? "right" : "left"}>
                <span class="username">{message.isMine ? username : message.username}</span> <br/>
                {message.body}
            </p>
            {/each}
        </div>
        {#if user_id !== undefined}
        <p class="connected-as">Connected as <b>{username}</b></p>
        <div class="input-wrapper">
            <textarea on:input={autoResize} on:keydown={handleKeyDown} placeholder="Votre message" rows="1" cols="0" maxlength="500"></textarea>
            <img src="send.svg" alt="">
        </div>
        {/if}
    </div>
    <div class="modal-wrapper" class:d-none={username !== ''}>
        <div class="body">
            <p class="title-xl">Username</p>
            <p class="subtitle">Please, enter your username</p>
            <form on:submit={handleSubmit}>
                <input bind:value={username_input} class="mb-md" placeholder="Username" type="text">
                <button type="submit" disabled={username_input === ''} class="btn">Confirm</button>
            </form>
        </div>
    </div>
    <div class="backdrop" class:d-none={username !== ''}></div>
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
        --border-radius: 0.625rem;
    }

    :global(*) {
        letter-spacing: var(--default-letter-spacing);
        font-family: 'Roboto', sans-serif;
        color: var(--dark);
        margin: 0;
        padding: 0;
        box-sizing: border-box;
    }

    :global(body) {
        background: var(--bg-gray);
    }

    .d-none {
        display: none;
    }

    .btn {
        border-radius: var(--border-radius);
        font-size: var(--font-size-sm);
        border: 0;
        padding: .5rem 1rem;
        color: var(--white);
        background: var(--dark);
        cursor: pointer;
    }

    .btn:disabled {
        opacity: var(--opacity-fading);
        cursor: default;
    }

    .mb-md {
        margin-bottom: 1rem;
    }

    .md-lg {
        margin-bottom: 5.06rem;
    }

    .modal-wrapper {
        position: absolute;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
        z-index: 2;
        width: 100%;
    }

    .modal-wrapper .body {
        background: var(--bg-gray);
        max-width: 900px;
        margin: auto;
        padding: 5rem;
        border-radius: var(--border-radius);
    }

    .modal-wrapper .body .desc {
        font-size: var(--font-size-md);
        opacity: var(--opacity-fading);
        margin-bottom: 2rem;
    }

    .backdrop {
        background-color: var(--dark);
        opacity: var(--opacity-fading);
        position: absolute;
        top: 0;
        left: 0;
        z-index: 1;
        height: 100vh;
        width: 100vw;
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
        margin-bottom: 2rem;
    }

    .connected-as {
        margin-top: 2rem;
        margin-bottom: 1rem;
        opacity: var(--opacity-fading);
    }

    .chat-wrapper {
        display: flex;
        flex-direction: column;
        max-height: 400px;
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
        border-radius: var(--border-radius);
    }
    
    .chat-wrapper .username {
        opacity:var(--opacity-fading);
    }

    .right {
        margin-left: auto;
    }

    .left {
        margin-right: auto;
    }

    .input-wrapper {
        display: flex;
    }

    .input-wrapper textarea {
        resize: none;
        background-color: var(--white);
        border-radius: var(--border-radius);
        padding: 1.02rem 1.81rem;
        font-size: var(--font-size-sm);
        letter-spacing: -0.02rem;
        font-weight: 400;
        flex: 1;
        gap: 1.29rem;
        outline: none;
        border: 0;
    }

    input {
        font-size: var(--font-size-sm);
        background-color: var(--white);
        padding: 1.02rem 1.81rem;
        border-radius: var(--border-radius);
        letter-spacing: -0.02rem;
        outline: none;
        border: 0;
        width: 100%;
    }

    input::placeholder {
        opacity: var(--opacity-fading);
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