<script lang="ts">
	import Input from "$lib/input.svelte";
	import { onMount } from "svelte";

    let numbers = $state([]);
    let length = $state(3);

    let gameWon = $state(false);

    let scores = $state([]);

    function getRandomInt() {
        return Math.floor(Math.random() * 10);
    }

    function getNewNumber() {
        numbers = [];
        for (let i = 0; i < length; i++) {
            numbers.push(getRandomInt());
        }
        console.log(numbers.join(""));
    }

    onMount(() => {
        getNewNumber();
    });

function getScore(guess: string[], correct: number[]): number {
    const _correct = [...correct];
    const _guess = [...guess];
    let score = 0;

    // First pass: correct value at correct index (2 points)
    for (let i = 0; i < _guess.length; i++) {
        const guessNum = parseInt(_guess[i]);
        if (_correct[i] !== undefined && guessNum === _correct[i]) {
            score += 2;
            _correct[i] = undefined;
            _guess[i] = undefined;
        }
    }

    // Second pass: correct value at wrong index (1 point)
    for (let i = 0; i < _guess.length; i++) {
        if (_guess[i] !== undefined) {
            const guessNum = parseInt(_guess[i]);
            const matchIndex = _correct.indexOf(guessNum);
            if (matchIndex !== -1) {
                score += 1;
                _correct[matchIndex] = undefined;
            }
        }
    }

    return score;
}


    function guessSubmit(value: string[]) {
        const score = getScore(value, numbers);
        scores.push({guess: value.join(""), score});
        if(score === length * 2) {
            gameWon = true;
        }
    }

    function resetGame() {
        gameWon = false;
        scores = [];
        getNewNumber();
    }

</script>

<!-- <div class="numbers">
    {#each numbers as number}
        <h2>{number}</h2>
    {/each}
</div> -->
<div class="page">
    <h1>Guessing Game</h1>
    
    <div class="scores">
        {#each scores as entry}
            <div class="score">
                <Input length={length} disabled value={entry.guess.split("")}/>
                <h3>-</h3>
                <h3>{entry.score}</h3>
            </div>
        {/each}
    </div>
    
    {#if !gameWon}
        <Input callback={guessSubmit} length={length}/>
    {:else}
        <h4>Correct! <br/> Took you {scores.length} tries</h4>
        <button on:click={resetGame}>Play Again</button>
    {/if}
</div>
    
<style>
    h2, h3, h4, h5, h6 {
        margin: 0;
    }
    h1 {
        font-size: 3rem;
        color: #333;
        margin-bottom: 1rem;
    }
    h3 {
        font-size: 1.8rem;
        color: #919191;
    }
    h4 {
        font-size: 1.5rem;
        color: #333;
        text-align: center;
    }
    button {
        padding: 0.75rem 1.25rem;
        font-size: 1.2rem;
        background-color: #007bff;
        color: white;
        border: none;
        border-radius: 0.25rem;
        cursor: pointer;
    }
    .page {
        display: flex;
        flex-direction: column;
        align-items: center;
        gap: 1.5rem;
    }

    .scores {
        display: flex;
        flex-direction: column;
        gap: .5rem;
    }

    .score {
        display: flex;
        align-items: center;
        gap: 1rem;
    }
</style>