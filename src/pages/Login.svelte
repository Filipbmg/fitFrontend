<script>
    import { navigate } from "svelte-routing";
    import { toast, Toaster } from "svelte-french-toast";
    import { onMount } from "svelte";

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

        const result = await response.json();
        if (!response.ok) {
            throw new Error("Login mislykkedes");
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

        const result = await response.json();
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
    <nav class="navbar">
        <h2>
            NotePal
        </h2>
    </nav>
    <div class="container">
        {#if showLoginForm}
            <form on:submit|preventDefault={handleLoginToast} class="form">
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
    .navbar {
        position: fixed;
        top: 0;
        left: 0;
        right: 0;
        background-color: lightgrey;
        border-bottom: 2px solid black;
        padding: 10px;
        z-index: 1000;
        display: flex;
        justify-content: space-between;
        align-items: center;
    }

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
        background-color: #1877f2;;
    }

    .submit-button:hover {
        background-color: #166fe5;
    }

    .toggle-button {
        padding: 10px;
        width: 70%;
        border: none;
        border-radius: 5px;
        font-size: 16px;
        font-weight: bold;
        color: #fff;
        background-color: #42b72a;;
        margin: auto;
    }

    .toggle-button:hover {
        background-color: #36a420;
    }
</style>