<script>
    import Searchbar from "../components/Searchbar.svelte";
    import MovieCard from "../components/MovieCard.svelte";
    import { onMount } from "svelte";

    var movieCards = [];
    var movieIds = {};
    var loading = false;
    var recommendations = [];
    onMount(async () => {
        const response = await fetch("http://localhost:8000/autocomplete");
        movieIds = await response.json();
    });
    function getMovieIds(movieTitle) {
        var movieIndex = movieIds[movieTitle];
        return movieIndex;
    }

    async function postMovieRecommendations(movieIndexes) {
        loading = true;
        var message;

        const response = await fetch("http://localhost:8000/movie/make_recom", {
            method: "POST",
            headers: {
                "Accept": "application/json",
                "Content-Type": "application/json"
            },
            body: JSON.stringify(movieIndexes),
        });
        message = await response.json();
        while (message["status"] == "inProgress") {
            const response = await fetch(
                "http://localhost:8000/status/" + message["jobId"]
            );
            message = await response.json();
        }
        return message["recommendation"];
    }

    async function makeRecommendations() {
        var recom_index = [];
        var i;
        for (i = 0; i < movieCards.length; i++) {
            var movIndx = getMovieIds(movieCards[i]);
            recom_index.push(movIndx);
        }
        var newMovies = await postMovieRecommendations(recom_index);

        for (i = 0; i < newMovies.length; i++) {
            recommendations = [...recommendations, newMovies[i]["title"]];
        }
        loading = false;
    }

    function clearRecommendations() {
        recommendations = [];
        movieCards = [];
    }
</script>

<Searchbar bind:movieTitles={movieCards} />
{#if loading == false}
    {#if recommendations.length < 1}
        {#if movieCards.length < 1}
            <div class="container">
                <h2>Filme hinzufügen, um Empfehlungen zu erhalen</h2>
            </div>
        {:else}
            <button
                class="btn waves-effect waves-light"
                type="submit"
                name="action"
                on:click={makeRecommendations}
                >Empfehlung
            </button>
            {#each movieCards as card}
                <MovieCard movieTitle={card} />
            {/each}
        {/if}
    {:else}
        <button
            class="btn waves-effect waves-dark"
            type="submit"
            name="action"
            on:click={clearRecommendations}>Clear</button
        >
        {#each recommendations as card}
            <MovieCard movieTitle={card} />
        {/each}
    {/if}
{:else}
    <div class="container">
        <img src="giphy.gif" alt="Making Recommendations" />
    </div>
{/if}

<style>
</style>
