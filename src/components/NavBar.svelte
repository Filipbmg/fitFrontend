
<nav class="navbar">
    <img src="/fitbot-logo.png" alt="FitBot" class="navbar-logo" />
    <slot></slot>
</nav>

<style>
    .navbar {
        position: fixed;
        top: 0;
        left: 0;
        right: 0;
        background-color: #8fbc39;
        border-bottom: 2px solid black;
        padding: 0 10px;
        z-index: 1000;
        display: flex;
        justify-content: space-between;
        align-items: center;
    }

    .navbar-logo {
        height: 160px;
        width: auto;
    }
</style>
