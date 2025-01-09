<script>
  import { Router, Route } from 'svelte-routing';
  import Login from "./pages/Login.svelte";
  import Home from './pages/Home.svelte';

</script>

<main>
<Router>
  <Route path="/" component={Login}/>
  <Route path="/Home" component={Home}/>
</Router>
</main>
