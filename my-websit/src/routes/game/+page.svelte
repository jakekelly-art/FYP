
<script>
  import { goto } from '$app/navigation';
  import { onMount } from 'svelte';
  import { fade } from 'svelte/transition';
  import GameCanvas from '#lib/game/GameCanvas.svelte';

  const charactersStorageKey = 'game-characters-v1';
  const selectedCharacterStorageKey = 'game-character-selected-v1';
  const legacyCharacterStorageKey = 'game-character-v1';

  /** @type {{ id: string, name: string, body: 'circle', face: 'circle' } | null} */
  let character = $state(null);
  let loaded = $state(false);

  onMount(() => {
    const savedCharacters = localStorage.getItem(charactersStorageKey);
    /** @type {{ id: string, name: string, body: 'circle', face: 'circle' }[]} */
    let characters = [];
    if (savedCharacters) {
      const parsedCharacters = JSON.parse(savedCharacters);
      if (
        !Array.isArray(parsedCharacters) ||
        parsedCharacters.some(
          (item) =>
            typeof item !== 'object' ||
            item === null ||
            typeof item.id !== 'string' ||
            !item.id ||
            typeof item.name !== 'string' ||
            !item.name.trim() ||
            item.body !== 'circle' ||
            item.face !== 'circle'
        )
      ) {
        throw new Error('Saved character list is invalid.');
      }

      characters = parsedCharacters;
    } else {
      const legacyCharacter = localStorage.getItem(legacyCharacterStorageKey);
      if (legacyCharacter) {
        const parsedCharacter = JSON.parse(legacyCharacter);
        if (
          typeof parsedCharacter !== 'object' ||
          parsedCharacter === null ||
          typeof parsedCharacter.name !== 'string' ||
          !parsedCharacter.name.trim() ||
          parsedCharacter.body !== 'circle' ||
          parsedCharacter.face !== 'circle'
        ) {
          throw new Error('Saved character data is invalid.');
        }

        characters = [
          {
            ...parsedCharacter,
            id: typeof parsedCharacter.id === 'string' ? parsedCharacter.id : crypto.randomUUID()
          }
        ];
        localStorage.setItem(charactersStorageKey, JSON.stringify(characters));
      }
    }

    const selectedId = localStorage.getItem(selectedCharacterStorageKey);
    character = characters.find((item) => item.id === selectedId) ?? characters[0] ?? null;
    if (!character) {
      void goto('/character-creator');
      return;
    }

    localStorage.setItem(selectedCharacterStorageKey, character.id);
    localStorage.setItem(legacyCharacterStorageKey, JSON.stringify(character));
    loaded = true;
  });

</script>

<div in:fade={{ duration: 250 }}>
  <section class="game-page" aria-label="Game room">
    <GameCanvas showCharacter={loaded} />
    {#if !loaded}
      <p class="status" role="status">Loading your character...</p>
    {/if}
  </section>
</div>

<style>
  .game-page {
    position: relative;
    width: min(800px, 100%);
    aspect-ratio: 1;
    margin-inline: auto;
    overflow: hidden;
    isolation: isolate;
  }

  .status {
    position: absolute;
    z-index: 1;
    top: 12px;
    left: 12px;
    margin: 0;
    padding: 12px 16px;
    background: rgb(3 1 26 / 76%);
    color: #f3f3ff;
  }

</style>
