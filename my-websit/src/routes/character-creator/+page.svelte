<script>
  import { goto } from '$app/navigation';
  import { onMount } from 'svelte';
  import { fade } from 'svelte/transition';
  import CharacterCreator from '#lib/components/CharacterCreator.svelte';

  const charactersStorageKey = 'game-characters-v1';
  const selectedCharacterStorageKey = 'game-character-selected-v1';
  const legacyCharacterStorageKey = 'game-character-v1';

  /**
   * @typedef {{ id: string, name: string, body: 'circle', face: 'circle' }} Character
   */

  /** @type {Character[]} */
  let characters = $state([]);
  /** @type {Character | null} */
  let character = $state(null);
  let creating = $state(true);
  let loaded = $state(false);

  onMount(() => {
    const savedCharacters = localStorage.getItem(charactersStorageKey);
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

        const migratedCharacter = {
          ...parsedCharacter,
          id: crypto.randomUUID()
        };
        characters = [migratedCharacter];
        localStorage.setItem(charactersStorageKey, JSON.stringify(characters));
      }
    }

    const selectedId = localStorage.getItem(selectedCharacterStorageKey);
    character = characters.find((item) => item.id === selectedId) ?? characters[0] ?? null;
    creating = character === null;
    loaded = true;
  });

  /** @param {Character} newCharacter */
  function saveCharacter(newCharacter) {
    const savedCharacter = {
      ...newCharacter,
      id: newCharacter.id || crypto.randomUUID()
    };
    const existingIndex = characters.findIndex((item) => item.id === savedCharacter.id);
    characters =
      existingIndex === -1
        ? [...characters, savedCharacter]
        : characters.map((item) => (item.id === savedCharacter.id ? savedCharacter : item));
    character = savedCharacter;
    creating = false;
    localStorage.setItem(charactersStorageKey, JSON.stringify(characters));
    localStorage.setItem(selectedCharacterStorageKey, savedCharacter.id);
    localStorage.setItem(legacyCharacterStorageKey, JSON.stringify(savedCharacter));
    void goto('/game');
  }

  function startNewCharacter() {
    character = null;
    creating = true;
  }

  /** @param {Character} characterToDelete */
  function deleteCharacter(characterToDelete) {
    if (!window.confirm(`Delete ${characterToDelete.name}? This cannot be undone.`)) return;

    characters = characters.filter((item) => item.id !== characterToDelete.id);
    localStorage.setItem(charactersStorageKey, JSON.stringify(characters));

    if (characters.length === 0) {
      character = null;
      creating = true;
      localStorage.removeItem(selectedCharacterStorageKey);
      localStorage.removeItem(legacyCharacterStorageKey);
      return;
    }

    character = characters[0];
    creating = false;
    localStorage.setItem(selectedCharacterStorageKey, character.id);
    localStorage.setItem(legacyCharacterStorageKey, JSON.stringify(character));
  }

</script>

{#if loaded}
  <div in:fade={{ duration: 250 }}>
    <CharacterCreator
      initialCharacter={creating ? null : character}
      hasSavedCharacters={characters.length > 0}
      onSave={saveCharacter}
      onNew={startNewCharacter}
      onDelete={deleteCharacter}
    />
  </div>
{:else}
  <div in:fade={{ duration: 250 }}>
    <p class="status" role="status">Loading character creator...</p>
  </div>
{/if}

<style>
  .status {
    color: #f3f3ff;
  }
</style>
