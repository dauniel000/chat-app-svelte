<script lang="ts">
    import { onMount, onDestroy, tick } from 'svelte';
    import { currentUser, pb } from './pocketbase';
    import { blur, fade, fly } from 'svelte/transition'


    let newMessage: string;
    let messages: any = [];
    let unsubscribe: () => void;

    let element: any

    


    const scrollToBottom = async (node: any, question: string) => {
      
   
      
        node.scroll({ top: node.scrollHeight,  behavior: question});

    }; 
  
    onMount(async () => {
      

      const resultList = await pb.collection('messages').getList(1, 50000, {
        sort: 'created',
        expand: 'user',
      });
      messages = resultList.items;
      

      unsubscribe = await pb
        .collection('messages')
        .subscribe('*', async ({ action, record }) => {
          if (action === 'create') {
            const user = await pb.collection('users').getOne(record.user);
            record.expand = { user };
            messages = [...messages, record];
            await tick()
            scrollToBottom(element, "smooth")

          }
          if (action === 'delete') {
            messages = messages.filter((m: any) => m.id !== record.id);
            await tick
            scrollToBottom(element, "smooth")

          }
        });
        await tick
        scrollToBottom(element, "auto")
    });
      onDestroy(() => {
      unsubscribe?.();
    });
  
    async function sendMessage() {
      const data = {
        text: newMessage,
        user: $currentUser?.id,
      };
      const createdMessage = await pb.collection('messages').create(data);
      newMessage = '';
    }

    async function deleteMessages(id: string) {
      await pb.collection('messages').delete(id);
    }


  </script>


  

  <div class="messages overflow-y-scroll overflow-x-hidden max-h-[70vh] skrol" bind:this={element}>
    
    {#each messages as message (message.id)}
      <div class="messg flex max-w-3xl md:w-96 mb-4" transition:fly={{ delay: 0, duration: 1000, x: 500, y: 0, opacity: 0}}>
        <img
          class="avatar"
          src={`https://api.dicebear.com/7.x/big-smile/svg?seed=sigma${message.expand?.user?.username}`}
          alt="avatar"
          width="100px"
        />
        <div class="relative w-full px-2">
          <small class="font-medium text-zinc-700">
            Wysłane przez <a href={`/user/${message.expand?.user?.username}`} class="text-blue-900">@{message.expand?.user?.username}</a>
          </small>
          <p class="msg-text text-xl font-medium">{message.text}</p>
          <button class="absolute text-red-300 right-2 top-0 cursor-pointer" on:click={() => deleteMessages(message.id)}>X</button>

        </div>
      </div>
    {/each}
  </div>
  
  
  <form on:submit|preventDefault={sendMessage} class="mt-3 bottom-3 w-full bg-zinc-950 shadow-lg md:w-96">
    <input placeholder="Wiadomość" type="text" bind:value={newMessage} class="bg-transparent border border-zinc-800 outline-none w-full px-4 py-3 placeholder:text-zinc-700 placeholder:transition-colors focus:placeholder:text-zinc-100 transition-colors rounded focus:border-zinc-600 font-medium"/>
    <button type="submit" class="w-full py-2 mt-2 bg-zinc-900">Wyślij</button>
  </form>