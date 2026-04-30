<script>
    import vida1 from "../../img/barra_de_vida_1.png";
    import vida2 from "../../img/barra_de_vida_2.png";
    import vida3 from "../../img/barra_de_vida_3.png";

    const imagens_vidas = [vida1, vida2, vida3];

    let { numVidas } = $props();
</script>

<div class="vidas">
    {#each imagens_vidas as imagemVida, numImagem}
        <img
            class="img-coracoes"
            src={imagemVida}
            alt="{numVidas} {numImagem}"
            style="background-size: 300px 175px; display: {numVidas ==
            numImagem + 1
                ? 'block'
                : 'none'}"
        />
    {/each}
</div>

<style>
    .vidas {
        width: 300px;
        height: 175px;
    }

    .img-coracoes {
        width: 300px;
        height: 175px;
    }
</style>
