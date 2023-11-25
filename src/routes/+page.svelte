<script lang="ts">
    import {goto} from '$app/navigation'

    function goToViewer() {
        goto('/viewer')
    }
    function goToAddMeal() {
        goto('/addmeal')
    }
    function goToCreateFood() {
        goto('/create')
    }
</script>

<main>
    <h1>Diet log</h1> <br><br>
    <button3 on:click={goToViewer}>Go to day viewer</button3> <br><br>
    <button3 on:click={goToAddMeal}>Add a meal to a day</button3><br><br>
    <button3 on:click={goToCreateFood}>Create a food type</button3>
</main>