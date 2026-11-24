<script>
  import { goto } from '$app/navigation';
  import { onMount } from 'svelte';
  import { fade } from 'svelte/transition';
  import CharacterPortrait from '#lib/components/CharacterPortrait.svelte';

  const charactersStorageKey = 'game-characters-v1';
  const selectedCharacterStorageKey = 'game-character-selected-v1';
  const legacyCharacterStorageKey = 'game-character-v1';

  /**
   * @typedef {{ id: string, name: string, body: 'circle', face: 'circle' }} Character
   */

  /** @type {Character[]} */
  let characters = $state([]);
  let loaded = $state(false);

  onMount(() => {
    const storedCharacters = localStorage.getItem(charactersStorageKey);
    if (storedCharacters) {
      const parsedCharacters = JSON.parse(storedCharacters);
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
        throw new Error('Saved character data is invalid.');
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
            id:
              typeof parsedCharacter.id === 'string' && parsedCharacter.id
                ? parsedCharacter.id
                : crypto.randomUUID()
          }
        ];
        localStorage.setItem(charactersStorageKey, JSON.stringify(characters));
      }
    }

    loaded = true;
  });

  /**
   * @param {Character} character
   * @param {'/game' | '/character-creator'} destination
   */
  function openCharacter(character, destination) {
    localStorage.setItem(selectedCharacterStorageKey, character.id);
    localStorage.setItem(legacyCharacterStorageKey, JSON.stringify(character));
    void goto(destination);
  }
</script>

<div in:fade={{ duration: 250 }}>
  <section class="characters-page" aria-label="Saved characters">

    {#if !loaded}
      <p role="status">Loading characters...</p>
    {:else if characters.length === 0}
      <div class="empty-state">
        <p>No characters yet.</p>
        <a href="/character-creator">Create your first character</a>
      </div>
    {:else}
      <div class="character-grid">
        {#each characters as character (character.id)}
          <article class="character-card">
            <div class="pet-preview">
              <CharacterPortrait label={`Character ${character.name}`} />
            </div>
            <div class="actions">
              <button type="button" onclick={() => openCharacter(character, '/game')} aria-label={`Play as ${character.name}`}>
                Play
              </button>
              <button type="button" onclick={() => openCharacter(character, '/character-creator')} aria-label={`Edit ${character.name}`}>
                Edit
              </button>
            </div>
          </article>
        {/each}
        <a class="create-card" href="/character-creator" aria-label="Create a new character">
          <span aria-hidden="true">+</span>
          <span>NEW</span>
        </a>
      </div>
    {/if}
  </section>
</div>

<style>
  .characters-page {
    width: min(900px, calc(100% - 32px));
    margin: 24px auto;
    color: #f3f3ff;
  }

  .character-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
    gap: 14px;
  }

  .character-card,
  .create-card {
    min-width: 0;
    min-height: 190px;
    border: 1px solid rgb(243 243 255 / 55%);
    background: rgb(243 243 255 / 4%);
  }

  .character-card {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: space-between;
    padding: 14px;
  }

  .pet-preview {
    display: grid;
    width: 100%;
    min-height: 120px;
    place-items: center;
  }

  .pet-preview :global(.portrait) {
    width: 90px;
  }

  .actions {
    display: flex;
    width: 100%;
    gap: 7px;
  }

  .actions button,
  .create-card {
    border: 1px solid #f3f3ff;
    clip-path: polygon(0 0, 94% 0, 100% 24%, 100% 100%, 6% 100%, 0 76%);
    color: #03011a;
    background: #f3f3ff;
    font: inherit;
  }

  .actions button {
    flex: 1;
    min-height: 34px;
    cursor: pointer;
  }

  .actions button:hover,
  .actions button:focus-visible,
  .create-card:hover,
  .create-card:focus-visible,
  .empty-state a:hover,
  .empty-state a:focus-visible {
    color: #f3f3ff;
    background: transparent;
  }

  .create-card {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    gap: 8px;
    text-decoration: none;
  }

  .create-card span:first-child {
    font-size: 2rem;
  }

  .empty-state a {
    display: inline-flex;
    justify-content: space-between;
    min-width: 240px;
    margin-top: 12px;
    padding: 10px 12px;
    border: 1px solid #f3f3ff;
    clip-path: polygon(0 0, 94% 0, 100% 24%, 100% 100%, 6% 100%, 0 76%);
    color: #03011a;
    background: #f3f3ff;
    text-decoration: none;
  }

  a:focus-visible,
  button:focus-visible {
    outline: 1px solid #f3f3ff;
    outline-offset: 2px;
  }

  @media (max-width: 520px) {
    .character-grid {
      grid-template-columns: repeat(2, minmax(0, 1fr));
    }
  }
</style>
