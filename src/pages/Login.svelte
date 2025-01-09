<script>
    import { navigate } from "svelte-routing";
    import { toast, Toaster } from "svelte-french-toast";
    import { onMount } from "svelte";
    import NavBar from "../components/NavBar.svelte";

    let showLoginForm = true;
    let email = "";
    let password = "";

    function toggle() {
        showLoginForm = !showLoginForm;
    }

    async function login() {
        const response = await fetch("http://localhost:8080/login", {
            method: "POST",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify({email, password}),
            credentials: "include",
        });

        if (!response.ok) {
            const errorData = await response.json().catch(() => ({}));
            throw new Error(errorData.message || "Login mislykkedes");
        }

        setTimeout(() => {
            navigate("/Home", { replace: true });
        }, 2000);
    }

    async function handleLoginToast() {
        await toast.promise(
            login(),
            {
                loading: "Logger ind...",
                success: "Succes!",
                error: "Login mislykkedes - Prøv igen",
            },
            {
                duration: 2000,
            },
        );
    }

    async function signUp() {
        const response = await fetch("http://localhost:8080/signup", {
            method: "POST",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify({email, password}),
        });
        
        if (!response.ok) {
            if (response.status === 409) {
                throw new Error("Emailen er allerede i brug");
            } else {
                throw new Error("Registrering mislykkedes");
            }
        }
        toggle();
    }

    async function handleSignUpToast() {
        await toast.promise(
            signUp(),
            {
                loading: "Opretter bruger...",
                success: "Success!",
                error: "Bruger oprettelse mislykkedes - Prøv igen",
            },
            {
                duration: 2000,
            },
        );
    }

    onMount(async () => {
        try {
            const response = await fetch("http://localhost:8080/auth/status", {
            credentials: "include"
            });
            const data = await response.json();

            if (data.isAuthenticated) {
                navigate("/Home", { replace: true });
            }
        } catch (err) {
            throw new Error("Fejl i session behandling");
        }
    })
</script>

<Toaster/>
<main>
    <NavBar/>
    <div class="container">
        {#if showLoginForm}
            <form  class="form" on:submit|preventDefault={handleLoginToast}>
                <label for="email">Email</label>
                <input type="email" bind:value={email} id="email" required placeholder="Email"/>
    
                <label for="password">Kodeord</label>
                <input type="password" bind:value={password} id="password" required placeholder="Kodeord"/>
    
                <button type="submit" class="submit-button">Log Ind</button>
                <button class="toggle-button" on:click={toggle}>Opret Bruger</button>
            </form>
        {:else}
            <form on:submit|preventDefault={handleSignUpToast} class="form">
                <label for="email">Email</label>
                <input type="email" bind:value={email} id="email" required placeholder="Email"/>
    
                <label for="password">Kodeord</label>
                <input type="password" bind:value={password} id="password" required placeholder="Kodeord"/>
    
                <button type="submit" class="submit-button">Opret Bruger</button>
                <button class="toggle-button" on:click={toggle}>Log Ind</button>
            </form>
        {/if}
    </div>
</main>


<style>
    .container {
        max-width: 365px;
        margin: auto;
        padding: 20px 20px;
        background-color: lightgrey;
        border-radius: 8px;
        text-align: center;
    }
    .form {
        display: flex;
        flex-direction: column;
        width: 100%;
    }

    .form input {
        margin-bottom: 10px;
        padding: 10px;
        border: 1px solid #ccc;
        border-radius: 5px;
        font-size: 16px;
        width: 100%;
        box-sizing: border-box;
    }

    .submit-button {
    margin-top: 10px;
    margin-bottom: 5px;
    padding: 10px;
    width: 100%;
    border: none;
    border-radius: 5px;
    font-size: 16px;
    font-weight: bold;
    color: #fff;
    background-color: #337ab7;
    }

    .submit-button:hover {
        background-color: #5a9bd6;
    }
    .toggle-button {
        padding: 10px;
        width: 70%;
        border: none;
        border-radius: 5px;
        font-size: 16px;
        font-weight: bold;
        color: #fff;
        background-color: #6b8e23;
        margin: auto;
    }

    .toggle-button:hover {
        background-color: #8fbc39;
    }
</style>