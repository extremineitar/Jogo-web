<script>
    import { onMount } from "svelte";
    
    let mouseX = $state(0);
    let mouseY = $state(0);

    function handleMouseMove(e) {
        mouseX = e.clientX;
        mouseY = e.clientY;
    }

    onMount(() => {
        window.addEventListener("mousemove", handleMouseMove);

        return () => {
            window.removeEventListener("mousemove", handleMouseMove);
        };
    });
</script>

<button
    title=""
    class="circulo"
    style="
        left: {mouseX}px;
        top: {mouseY}px;
    "
></button>

<style>
    .circulo {
        position: fixed;
        width: 40px;
        height: 40px;
        border-radius: 50%;
        border: 2px solid white;
        background: transparent;

        transform: translate(-50%, -50%);

        pointer-events: none;
        cursor: none;
        z-index: 3;
    }
</style>
