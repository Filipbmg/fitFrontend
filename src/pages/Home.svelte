<script>
  import LogoutButton from "../components/LogoutButton.svelte";
  import NavBar from "../components/NavBar.svelte";
  import { toast, Toaster } from "svelte-french-toast";
  import { onMount } from "svelte";
  import { navigate } from "svelte-routing";

  let backendData = null;
  let activeStep = 0;
  let adviceText = "";
  let cursorVisible = false;
  let showDownloadAndDeleteButtons = false;
  let isLoading = false;
  let formData = {
    gender: "",
    age: "",
    goal: "",
    experience: "",
    trainingFreq: ""
  };

  async function typeAdvice(adviceInput) {
    adviceText = "";
    cursorVisible = true;

    for (let char of adviceInput) {
      adviceText += char;
      await new Promise(resolve => setTimeout(resolve, 25));
    }

    cursorVisible = false;
    showDownloadAndDeleteButtons = true;
  };
  
  async function submitForm() {
    try {
      const response = await fetch("http://localhost:8080/generate-routine", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        credentials: "include",
        body: JSON.stringify(formData)
      })

      if (!response.ok) {
        activeStep = 0;
        throw new Error("Fejl ved Generering af træningsplan");
      }
      
      backendData = await response.json();
      typeAdvice(backendData.advice)
    } catch (err) {
      toast.error(err.message || "Fejl ved generering af træningsplan")
    }  
  };  



  async function handleSubmit() {
    isLoading = true; 

    try {
        await submitForm();
    } catch (error) {
        toast.error("Indsendelse af data fejlede - prøv igen");
    } finally {
        isLoading = false;
    }
  };

  async function deleteRoutine() {
    try {
      const response = await fetch("http://localhost:8080/routine", {
        method: "DELETE",
        credentials: "include"
      });

      if (!response.ok) {
        throw new Error("Fejl ved sletning af træningsplan");
      }

      backendData = null;
      activeStep = 0;
    } catch (err) {
      toast.error(err.message || "Fejl ved sletning af træningsplan");
    }  
  }

  onMount(async () => {
    try {
      const response = await fetch("http://localhost:8080/auth/status", {
        credentials: "include"
      });

      const data = await response.json();
      if (!data.isAuthenticated) {
        navigate("/", { replace: true });
      }

      const routineResponse = await fetch("http://localhost:8080/routine", {
        credentials: "include",
      });
      
      if (routineResponse.ok) {
        backendData = await routineResponse.json();
        adviceText = backendData.advice
        showDownloadAndDeleteButtons = true;
      } else if (routineResponse.status !== 404) {
        throw new Error("Fejl ved hentning af eksisterende træningsplan");
      }
    } catch (err) {
      throw new Error(err.message || "Fejl ved behandling af session data")
    }
  })
</script>

<Toaster/>
<main>
  <NavBar>
    <LogoutButton/>
  </NavBar>

  {#if isLoading}
    <div class="loading-container">
      <div class="spinner"></div>
      <p>Din træningsplan genereres - Vent venligst</p>
    </div>



  {:else if !backendData}
    <form class="form" on:submit|preventDefault={handleSubmit}>
      {#if activeStep === 0}
        <h2>Hvad er dit køn?</h2>
        <div class="button-group">
          <button type="button" on:click={() => { formData.gender = "Køn: Mand"; activeStep++; }}>Mand</button>
          <button type="button" on:click={() => { formData.gender = "Køn: Kvinde"; activeStep++; }}>Kvinde</button>
        </div>
        <input type="hidden" name="gender" value={formData.gender} />
    


      {:else if activeStep === 1}
        <h2>Hvad er din alder?</h2>
        <div class="button-group">
          <button type="button" on:click={() => { formData.age = "Alder: 18 - 29"; activeStep++; }}>18 - 29</button>
          <button type="button" on:click={() => { formData.age = "Alder: 30 - 39"; activeStep++; }}>30 - 39</button>
          <button type="button" on:click={() => { formData.age = "Alder: 40 - 49"; activeStep++; }}>40 - 49</button>
          <button type="button" on:click={() => { formData.age = "Alder: 50+"; activeStep++; }}>50+</button>
        </div>
        <button type="button" class="back-button" on:click={() => { activeStep--; }}>← Tilbage</button>
        <input type="hidden" name="age" value={formData.age} />


    
      {:else if activeStep === 2}
        <h2>Hvad er dit mål?</h2>
        <div class="button-group">
          <button type="button" on:click={() => { formData.goal = "Mål: Primære mål er at bygge muskler, sekundære mål er at blive stærkere"; activeStep++; }}>Mit primære mål er at bygge muskler, men jeg vil også gerne være stærkere</button>
          <button type="button" on:click={() => { formData.goal = "Mål: Primære mål er at blive stærkere, sekundære mål er at bygge muskler"; activeStep++; }}>Mit primære mål er at blive stærkere, men jeg vil også gerne have større muskler</button>
          <button type="button" on:click={() => { formData.goal = "Mål: Primære mål er at tabe fedt, sekundære mål er at bygge muskler"; activeStep++; }}>Mit primære mål er at tabe mig, men jeg vil også gerne bygge muskler</button>
        </div>
        <button type="button" class="back-button" on:click={() => { activeStep--; }}>← Tilbage</button>
        <input type="hidden" name="goal" value={formData.goal} />


    
      {:else if activeStep === 3}
        <h2>Hvor længe har du trænet regelmæssigt?</h2>
        <p>I træningscentret minimum 2-3 gange om ugen</p>
        <div class="button-group">
          <button type="button" on:click={() => { formData.experience = "Erfaring: Begynder"; activeStep++; }}>Under 1 - 2 år</button>
          <button type="button" on:click={() => { formData.experience = "Erfaring: Lettere øvet"; activeStep++; }}>2 - 5 år</button>
          <button type="button" on:click={() => { formData.experience = "Erfaring: Øvet"; activeStep++; }}>Over 5 år</button>
        </div>
        <button type="button" class="back-button" on:click={() => { activeStep--; }}>← Tilbage</button>
        <input type="hidden" name="experience" value={formData.experience} />
    


      {:else if activeStep === 4}
        <h2>Hvor mange dage om ugen har du tid/lyst til at træne?</h2>
        <div class="button-group">
          <button type="submit" on:click={() => { formData.trainingFreq = "Træningsdage per uge: 3" }}>3</button>
          <button type="submit" on:click={() => { formData.trainingFreq = "Træningsdage per uge: 4" }}>4</button>
          <button type="submit" on:click={() => { formData.trainingFreq = "Træningsdage per uge: 5" }}>5</button>
          <button type="submit" on:click={() => { formData.trainingFreq = "Træningsdage per uge: 6" }}>6</button>
        </div>
        <button type="button" class="back-button" on:click={() => { activeStep--; }}>← Tilbage</button>
        <input type="hidden" name="trainingFreq" value={formData.trainingFreq} />
      {/if}
    </form>



  {:else}
    <div class="response-container">
      <h2>Træningsplan og Råd</h2>
      <div class="text-container">
        <p class="typing-effect">
          {adviceText}
          {#if cursorVisible}
          <span class="cursor">|</span>
          {/if}
        </p>
      </div>
      
      {#if showDownloadAndDeleteButtons}
        <a
          href={`data:application/vnd.openxmlformats-officedocument.spreadsheetml.sheet;base64,${backendData.excelFile}`}
          download="træningsplan.xlsx"
          class="download-button"
          >Download Træningsplan</a
        >
        <button class="delete-button" on:click|preventDefault={deleteRoutine}>
          Slet Træningsplan
        </button>
        
      {/if}
    </div>
  {/if}
</main>

<style>
  .delete-button {
    margin-top: 16px;
    padding: 12px 24px;
    background-color: #cd1c1c;
    color: white;
    border: none;
    border-radius: 5px;
    text-decoration: none;
    font-weight: bold;
    font-size: 16px;
    cursor: pointer;
    transition: background-color 0.3s ease, transform 0.2s ease;
    display: inline-block;
  }

  .delete-button:hover {
    background-color: #b21818;
    transform: scale(1.05);
  }


  .loading-container {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    height: 100%;
    text-align: center;
    padding: 20px;
  }

  .spinner {
    width: 50px;
    height: 50px;
    border: 5px solid #e3e3e3;
    border-top: 5px solid #8fbc39;
    border-radius: 50%;
    animation: spin 1s linear infinite;
    margin-bottom: 10px;
  }

  @keyframes spin {
    0% {
      transform: rotate(0deg);
    }
    100% {
      transform: rotate(360deg);
    }
  }

  .response-container {
    display: flex;
    flex-direction: column;
    align-items: center;
  }

  .text-container {
    max-width: 600px;
    margin: 16px auto;
    text-align: left;
    line-height: 1.5;
    word-wrap: break-word;
    overflow-wrap: break-word;
    white-space: normal;
  }

  .typing-effect {
    font-family: Arial, sans-serif;
    white-space: pre-wrap;
    text-align: left;
    margin: 16px 0;
  }

  .cursor {
    display: inline-block;
    width: 1px;
    background-color: transparent;
    animation: blink 0.8s steps(2, start) infinite;
  }

  .download-button {
    margin-top: 16px;
    padding: 12px 24px;
    background-color: #6b8e23;
    color: white;
    border: none;
    border-radius: 5px;
    text-decoration: none;
    font-weight: bold;
    font-size: 16px;
    cursor: pointer;
    transition: background-color 0.3s ease, transform 0.2s ease;
    display: inline-block;
  }

  .download-button:hover {
    background-color: #8fbc39;
    transform: scale(1.05);
  }

  @keyframes blink {
    0%,
    100% {
      background-color: transparent;
    }
    50% {
      background-color: black;
    }
  }

  .form h2 {
    margin-bottom: 0px;
  }

  .form p {
    margin-top: 0px;
    margin-bottom: 16px;
    text-align: center;
  }

  .form {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 16px;
  }

  .button-group {
    display: flex;
    flex-direction: column;
    gap: 12px;
    align-items: center;
  }

  button {
    display: block;
    padding: 12px 20px;
    border: none;
    border-radius: 5px;
    font-size: 16px;
    font-weight: bold;
    text-align: center;
    cursor: pointer;
    background-color: #6b8e23;
    color: white;
    transition: background-color 0.3s ease, transform 0.2s ease;
    word-wrap: break-word;
    white-space: normal;
    min-width: 192px;
    max-width: 320px;
  }

  button:hover {
    background-color: #8fbc39;
    transform: scale(1.05);
  }

  .back-button {
    background-color: transparent;
    color: black;
    min-width: 192px;
  }

  .back-button:hover {
    background-color: white;
    transform: scale(1.05);
  }
</style>
  
