<script>
    import { onMount } from 'svelte';
    import { Heading, P, Label, Input, Button } from 'flowbite-svelte';
    import { account, databases } from '$lib/appwrite';
    import { Query } from 'appwrite';
    import { PUBLIC_HEADPATDB, PUBLIC_COLLECTION_USERDATA } from '$env/static/public';

    let user;
    let returningUser = true;
    let username = '';

    onMount(async () => {
        const urlParams = new URLSearchParams(window.location.search);
        const secret = urlParams.get('secret');
        const userId = urlParams.get('userId');

        user = await account.createSession(userId, secret);
        console.log(user);

        const userdata = await databases.listDocuments(PUBLIC_HEADPATDB, PUBLIC_COLLECTION_USERDATA, [
            Query.equal('userID', user.userId),
            Query.limit(1),
        ]);

        if (userdata.documents.length === 0) {
            returningUser = false;
        }
    });
</script>

<svelte:head>
    <title>Give Headpats</title>
</svelte:head>

<div class="flex justify-center items-center">
    <div class="bg-white dark:bg-slate-900 rounded-lg p-6 max-w-lg sm:min-w-[50%] min-w-[90%]">
        {#if returningUser}
            <Heading class="mb-3" tag="h2" customSize="text-4xl font-extrabold ">Logged in successfully.</Heading>
            <P class="mb-6">Welcome back.</P>
            <a data-sveltekit-reload href="/"><Button color="dark">Go home</Button></a>
        {:else}
            <Heading tag="h2" customSize="text-4xl font-extrabold ">Welcome!</Heading>
            <P class="mb-3">It's nice to meet you! Let's get you set up.</P>
            <div class="mb-6">
                <Label for="username" class="block mb-2">Username</Label>
                <Input id="username" placeholder="areallycuteperson" bind:value={username} />
            </div>

            <Button
                color="blue"
                on:click={async () => {
                    let accountResponse = await fetch('/api/validate', {
                        method: 'POST',
                        body: JSON.stringify({
                            username: username.toLowerCase(),
                        }),
                        headers: {
                            'content-type': 'application/json',
                        },
                    });
                    accountResponse = await accountResponse.json();

                    if (accountResponse.status !== 200) return alert(`${accountResponse.message}`);

                    await fetch('/api/setupaccount', {
                        method: 'POST',
                        body: JSON.stringify({
                            user: username.toLowerCase(),
                            userID: user.userId,
                        }),
                        headers: {
                            'content-type': 'application/json',
                        },
                    });

                    location.reload();
                }}>Finish</Button>
        {/if}
    </div>
</div>
