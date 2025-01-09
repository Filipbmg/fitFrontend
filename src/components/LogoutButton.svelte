<script>
    import { navigate } from "svelte-routing";

    async function logOut() {
        try {
            const response = await fetch("http://localhost:8080/logout", {
                method: "POST",
                credentials: "include"
            });

            if (!response.ok) {
                throw new Error("Kunne ikke logge ud");
            }

            navigate("/");
        } catch (err) {
            throw new Error("Fejl ved logout: " + err.message);
        }
    }
</script>

<button on:click={logOut} class="logout-button">
    Log Ud
</button>

<style>
    .logout-button {
        display: flex;
        align-items: center;
        justify-content: center;
        width: 120px;
        height: 40px;
        background-color: #cd1c1c;
        color: white;
        border: none;
        border-radius: 5px;
        cursor: pointer;
        transition: background-color 0.3s ease, transform 0.2s ease;
    }

    .logout-button:hover {
        background-color: #b21818;
        transform: scale(1.05);
    }
</style>