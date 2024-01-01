<script lang="ts">
    import { blur, fade, fly } from "svelte/transition";
    import Messages from "./Messages.svelte";
import { currentUser, pb } from "./pocketbase";

    let username: string
    let password: string

    async function login() {
        await pb.collection('users').authWithPassword(username, password)
    }

    async function signUp() {
        try {
            const data = {
            username,
            password,
            passwordConfirm: password,
            name: 'Mariusz'
        }
        const createdUser = await pb.collection('users').create(data)
        await login()
        } catch (err) {

        }
    }

    function signOut () {
        pb.authStore.clear()
    }


</script>


{#if $currentUser}
    <div class="w-full flex flex-col items-center" transition:fade>
        <div class="flex justify-between md:w-96">
            <button on:click={signOut} class="font-bold" >Wyloguj</button>
            <img
            class="avatar"
            src={`https://api.dicebear.com/7.x/big-smile/svg?seed=sigma${$currentUser.username}` }
            alt="avatar"
            width="60px"
            
          />
        </div>
    </div>
    
    <div class="w-full min-h-full relative flex flex-col items-center">
    <Messages/>

    </div>

{:else}
    <div class="flex flex-col items-center ">
        <form on:submit|preventDefault class="flex flex-col min-h-[90vh] justify-center transition-colors md:w-96">
            <h1 class="text-xl font-bold mb-4">Zaloguj</h1>
            <input type="text" placeholder="Nazwa użytkownika" bind:value={username} class="w-full px-5 py-3 bg-transparent border border-zinc-800 focus:border-zinc-600 placeholder:text-zinc-700 focus:placeholder:text-zinc-200 text-zinc-200 focus:placeholder:transition-colors transition-colors rounded outline-none focus:outline-none">
            <input type="password" placeholder="Hasło" bind:value={password} class="w-full px-5 py-3 bg-transparent border border-zinc-800 focus:border-zinc-600 placeholder:text-zinc-700 focus:placeholder:text-zinc-200 text-zinc-200 focus:placeholder:transition-colors transition-colors rounded outline-none focus:outline-none mt-5">
            <button on:click={login} class="mt-3 bg-zinc-900 py-3 hover:bg-zinc-800 transition-colors rounded font-bold uppercase">zaloguj się</button>
            <button on:click={signUp} class="mt-5 bg-blue-900 py-3 hover:bg-blue-800 transition-colors rounded font-bold uppercase">zarejestruj</button>
        </form>
    </div>
{/if}