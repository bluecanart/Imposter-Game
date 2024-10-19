<script lang="ts">
  import { onMount } from "svelte";
  import words from "./data/words";

  // Define types for categories and words
  type WordGrid = string[];
  let selectedCategory: string = '';
  let gridWords: WordGrid = [];
  let numPlayers: number = 2; // Default to 2 players
  let correctWordIndex: number = -1;
  let imposterIndex: number = -1;
  let revealedIndex: number = -1;
  let revealedPlayerIndex: number = -1;

  function getRandomFromArray<T>(array: T[]): T {
    return array[Math.floor(Math.random() * array.length)];
  }

  // Function to start the game
  function startGame(): void {
    selectedCategory = getRandomFromArray(Object.keys(words));
    gridWords = shuffleArray(words[selectedCategory]).slice(0, 16);
    assignImposter();
  }

  // Function to shuffle array
  function shuffleArray(array: string[]): string[] {
    return array.sort(() => Math.random() - 0.5);
  }

  // Function to assign imposter and words
  function assignImposter(): void {
    correctWordIndex = Math.floor(Math.random() * gridWords.length);
    imposterIndex = Math.floor(Math.random() * numPlayers);
    revealedIndex = -1;
    revealedPlayerIndex = -1;
  }

  // Reveal word for player
  function toggleReveal(index: number): void {
    console.log({
        index,
        revealedIndex,
        revealedPlayerIndex,
        imposterIndex,
    });
    if (revealedIndex === -1) {
      // If the word is not already revealed
      revealedPlayerIndex = index;
      if (index === imposterIndex) {
        // Imposter reveals nothing
        revealedIndex = -2;
      } else {
        // Non-imposter reveals the correct word
        revealedIndex = correctWordIndex;
      }
    } else {
      // If the word is already revealed, hide it again
      revealedIndex = -1;
      revealedPlayerIndex = -1;
    }
  }

  // Handle game reset
  function resetGame(): void {
    startGame();
  }

  // Handle number of players
  function handlePlayerChange(event: Event): void {
    const target = event.target as HTMLSelectElement;
    numPlayers = +target.value;
    resetGame();
  }

  // Helper function to determine the CSS class for highlighting
  function getHighlightClass(wordIndex: number, revealedWordIndex: number): string {
    return revealedWordIndex === wordIndex ? "bg-green-400" : "bg-white";
  }

  // Start game on mount
  onMount(() => {
    startGame();
  });
</script>

<!-- UI for the game -->
<div class="p-4 max-w-4xl mx-auto font-sans">
  <!-- Category display -->
  <h1 class="text-2xl font-bold mb-4 text-center">
    Category: {selectedCategory}
  </h1>

  <!-- Top bar with dropdown and reset button -->
  <div class="flex justify-between items-center mb-4">
    <!-- Dropdown for player selection -->
    <div>
      <label for="numPlayers" class="block text-lg font-semibold mb-1"
        >Number of players:</label
      >
      <select
        id="numPlayers"
        on:change={handlePlayerChange}
        class="block w-24 p-2 border rounded-md"
      >
        {#each Array(6).fill(0) as _, i}
          <option value={i + 2}>{i + 2}</option>
        {/each}
      </select>
    </div>

    <!-- Reset button, floating to the right -->
    <div>
      <button
        on:click={resetGame}
        class="py-2 px-4 bg-red-500 hover:bg-red-700 text-white rounded-md"
      >
        Reset Game
      </button>
    </div>
  </div>

  <!-- 5x5 word grid -->
  <div class="grid grid-cols-4 gap-4">
    {#each gridWords as word, wordIndex}
      <div
        class="grid-item p-4 border rounded text-center text-lg font-semibold {getHighlightClass(
          wordIndex, revealedIndex
        )}"
      >
        {word}
      </div>
    {/each}
  </div>

  <!-- Buttons for players to reveal their word -->
  <div class="flex flex-wrap justify-start mt-6">
    {#each Array(numPlayers) as _, playerIndex}
      <button
        on:click={() => toggleReveal(playerIndex)}
        class="py-2 px-4 bg-blue-500 hover:bg-blue-700 text-white rounded-md m-2"
        disabled={revealedPlayerIndex !== -1 &&
          playerIndex != revealedPlayerIndex}
      >
        {#if revealedPlayerIndex === playerIndex}
          Hide Word
        {:else}
          Reveal Player {playerIndex + 1}
        {/if}
      </button>
    {/each}
  </div>
</div>

<style global lang="postcss">
  @tailwind utilities;
  @tailwind components;
  @tailwind base;
</style>