<!--
  - Header.svelte
  - Copyright (c) 2023 Jack Chakany <jack@chaker.net>
  -
  - This program is free software: you can redistribute it and/or modify
  - it under the terms of the GNU Affero General Public License as published by
  - the Free Software Foundation, either version 3 of the License, or
  - (at your option) any later version.
  -
  - This program is distributed in the hope that it will be useful,
  - but WITHOUT ANY WARRANTY; without even the implied warranty of
  - MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
  - GNU Affero General Public License for more details.
  -
  - You should have received a copy of the GNU Affero General Public License
  - along with this program.  If not, see <http://www.gnu.org/licenses/>.
  -->

<script lang="ts">
    import {NDK, user} from "$lib/stores"
    import { Avatar } from "@nostr-dev-kit/ndk-svelte-components"
    import { NDKNip07Signer } from "@nostr-dev-kit/ndk";
    import { page} from "$app/stores"
    $: pagePath = $page.url.pathname

    $: $user?.fetchProfile()
    async function login() {
        try {
            const signer = new NDKNip07Signer();
            $NDK.signer = signer;
        } catch (e) {
            console.error(e)
            alert("error with signer; check to see if you have a signer extension enabled")
        }
        const user = await $NDK.signer?.user()
        if (!user) {
            alert("no pubkey provided, try again")
            return
        }
        const myUser = (await user);
        localStorage.setItem("pubkey", myUser.npub)
        myUser.ndk = $NDK;
        // @ts-ignore
        $user = myUser
    }
</script>

<div class="flex mx-5 my-5">
    <div class="flex gap-3 my-auto">
        <a class="font-semibold" href="/" target="_self">Zapper News</a>
        <div class="flex gap-2">
            <a class="{pagePath == `/` ? `font-semibold` : ``}" href="/">top</a>
            <a class="{pagePath == `/recent` ? `font-semibold` : ``}" href="/recent">recent</a>
        </div>
    </div>
    <div class="flex ml-auto my-auto">
        <a class="my-auto mr-5 cursor-pointer" href="/post" target="_self">Post</a>
        <span class="flex my-auto gap-2">
            {#if $user}
                <Avatar class="rounded-full w-12" ndk={$NDK} npub={$user.npub} />
                <a on:click={() => { localStorage.clear()
                $user = null} } class="cursor-pointer my-auto">Logout</a>
            {:else}
                <span class="cursor-pointer" on:click={login}>Login</span>
            {/if}
        </span>
    </div>
</div>