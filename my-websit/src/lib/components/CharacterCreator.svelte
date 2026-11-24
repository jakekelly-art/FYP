<script>
  import circleBody from '#lib/assets/CIRCLE_BODY.png';
  import circleFace from '#lib/assets/FACE_CIRCLE_1.png';
  import paintStroke from '#lib/assets/White-Paint-Brush-Stroke-PNG-Clipart-Background-3091116128.png';
  import CharacterPortrait from '#lib/components/CharacterPortrait.svelte';

  /**
   * @typedef {{ id: string, name: string, body: 'circle', face: 'circle' }} Character
   */

  /**
   * @type {{
   *   initialCharacter?: Character | null,
   *   hasSavedCharacters?: boolean,
   *   onSave: (character: Character) => void,
   *   onNew: () => void,
   *   onDelete: (character: Character) => void
   * }}
   */
  let {
    initialCharacter = null,
    hasSavedCharacters = false,
    onSave,
    onNew,
    onDelete
  } = $props();

  let name = $state('');
  /** @type {'circle'} */
  let body = $state('circle');
  /** @type {'circle'} */
  let face = $state('circle');
  /** @type {'body' | 'face' | 'finish'} */
  let activeCategory = $state('body');

  $effect(() => {
    name = initialCharacter?.name ?? '';
  });

  /** @param {SubmitEvent} event */
  function submit(event) {
    event.preventDefault();
    const trimmedName = name.trim();
    if (!trimmedName) return;

    onSave({
      id: initialCharacter?.id ?? '',
      name: trimmedName,
      body,
      face
    });
  }
</script>

<section class="creator" aria-label="Character creator">
  <div class="creator-layout">
    <div class="preview">
      <p class="eyebrow">PREVIEW</p>
      <img class="paint-stroke" src={paintStroke} alt="" aria-hidden="true" />
      <div class="avatar">
        <CharacterPortrait />
      </div>
    </div>

    <form onsubmit={submit}>
      {#if initialCharacter || hasSavedCharacters}
        <a class="back-link" href="/game">Back to game</a>
      {/if}

      <div class="tabs" aria-label="Character part">
        <button
          type="button"
          aria-label="Body"
          title="Body"
          class:active={activeCategory === 'body'}
          aria-pressed={activeCategory === 'body'}
          onclick={() => (activeCategory = 'body')}
        >Body</button>
        <button
          type="button"
          aria-label="Face"
          title="Face"
          class:active={activeCategory === 'face'}
          aria-pressed={activeCategory === 'face'}
          onclick={() => (activeCategory = 'face')}
        >Face</button>
        <button
          type="button"
          aria-label="Finish"
          title="Finish"
          class:active={activeCategory === 'finish'}
          aria-pressed={activeCategory === 'finish'}
          onclick={() => (activeCategory = 'finish')}
        >Finish</button>
      </div>

      {#if activeCategory === 'body'}
        <button
          class="option"
          class:active={body === 'circle'}
          type="button"
          aria-pressed={body === 'circle'}
          onclick={() => (body = 'circle')}
        >
          <span class="option-image body-option"><img src={circleBody} alt="" /></span>
          <span>Circle body</span>
        </button>
        <button class="continue" type="button" onclick={() => (activeCategory = 'face')}>
          Continue to face
        </button>
      {:else if activeCategory === 'face'}
        <button
          class="option"
          class:active={face === 'circle'}
          type="button"
          aria-pressed={face === 'circle'}
          onclick={() => (face = 'circle')}
        >
          <span class="option-image face-option"><img src={circleFace} alt="" /></span>
          <span>Circle face</span>
        </button>
        <button class="continue" type="button" onclick={() => (activeCategory = 'finish')}>
          Finish character
        </button>
      {:else}
        <label class="name-field">
          <span class="eyebrow">NAME</span>
          <input bind:value={name} maxlength="24" placeholder="Enter a name" required />
        </label>

        <div class="actions">
          {#if initialCharacter}
            <button class="delete" type="button" onclick={() => onDelete(initialCharacter)}>
              Delete
            </button>
          {/if}
          {#if hasSavedCharacters}
            <button class="new" type="button" onclick={onNew}>New</button>
          {/if}
          <button class="save" type="submit">Save character</button>
        </div>
      {/if}
    </form>
  </div>
</section>

<style>
  .creator {
    width: min(900px, calc(100% - 32px));
    margin: 24px auto;
    color: #f3f3ff;
  }

  p {
    margin: 0;
  }

  .eyebrow {
    color: #f3f3ff;
    font-size: 0.75rem;
    letter-spacing: 0.12em;
  }

  .back-link {
    align-self: flex-end;
    padding: 9px 13px;
    border: 1px solid #f3f3ff;
    clip-path: polygon(0 0, 94% 0, 100% 24%, 100% 100%, 6% 100%, 0 76%);
    color: #03011a;
    background: #f3f3ff;
    text-decoration: none;
  }

  .back-link:hover,
  .back-link:focus-visible {
    color: #f3f3ff;
    background: transparent;
  }

  .creator-layout {
    display: grid;
    grid-template-columns: minmax(0, 1fr) minmax(260px, 0.8fr);
    gap: 24px;
    padding-top: 20px;
  }

  .preview {
    position: relative;
    display: flex;
    min-height: 430px;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    overflow: hidden;
    border: 1px solid rgb(243 243 255 / 55%);
  }

  .preview > .eyebrow {
    align-self: flex-start;
    margin: 14px 16px auto;
    font-size: 1rem;
  }

  .avatar {
    position: relative;
    z-index: 1;
    width: min(40%, 180px);
    aspect-ratio: 1244 / 1534;
  }

  .paint-stroke {
    position: absolute;
    top: 45%;
    left: 50%;
    width: 75%;
    height: auto;
    opacity: 1;
    pointer-events: none;
    transform: translate(-50%, -50%) rotate(180DEG);
  }

  form {
    display: flex;
    min-width: 0;
    flex-direction: column;
    gap: 14px;
  }

  .back-link {
    margin-bottom: 2px;
  }

  .new,
  .delete {
    min-height: 36px;
    padding: 7px 10px;
    border: 1px solid #f3f3ff;
    clip-path: polygon(0 0, 94% 0, 100% 24%, 100% 100%, 6% 100%, 0 76%);
    color: #03011a;
    background: #f3f3ff;
    font: inherit;
    cursor: pointer;
  }

  .tabs {
    display: flex;
    gap: 5px;
  }

  .tabs button,
  .option,
  .save,
  .continue {
    min-height: 42px;
    border: 1px solid #f3f3ff;
    clip-path: polygon(0 0, 94% 0, 100% 24%, 100% 100%, 6% 100%, 0 76%);
    color: #03011a;
    background: #f3f3ff;
    font: inherit;
    cursor: pointer;
  }

  .tabs button {
    display: flex;
    flex: 1;
    min-height: 58px;
    padding: 9px;
    align-items: center;
    justify-content: center;
  }

  .tabs button:not(.active),
  .option:not(.active) {
    opacity: 0.72;
  }

  .option {
    display: flex;
    align-items: center;
    gap: 12px;
    min-height: 76px;
    padding: 9px;
    text-align: left;
  }

  .option-image {
    display: grid;
    flex: 0 0 54px;
    width: 54px;
    height: 54px;
    place-items: center;
    border: 1px solid rgb(3 1 26 / 35%);
  }

  .body-option img {
    width: 26px;
    height: 42px;
    object-fit: contain;
  }

  .face-option img {
    width: 48px;
    height: 48px;
    object-fit: contain;
  }

  .name-field {
    display: grid;
    gap: 7px;
    margin-top: auto;
  }

  .name-field input {
    width: 100%;
    min-height: 42px;
    padding: 9px 11px;
    border: 1px solid #f3f3ff;
    clip-path: polygon(0 0, 98% 0, 100% 18%, 100% 100%, 2% 100%, 0 82%);
    color: #03011a;
    background: #f3f3ff;
    font: inherit;
  }

  .save {
    padding: 9px 13px;
    font-weight: 700;
  }

  .actions {
    display: flex;
    gap: 7px;
  }

  .actions .save {
    flex: 1;
  }

  .continue {
    display: flex;
    align-items: center;
    justify-content: flex-start;
    width: 100%;
    min-height: 42px;
    padding: 9px 13px;
    font-weight: 700;
  }

  .back-link:hover,
  .back-link:focus-visible,
  .tabs button:hover,
  .option:hover,
  .option:focus-visible,
  .new:hover,
  .new:focus-visible,
  .delete:hover,
  .delete:focus-visible,
  .continue:hover,
  .continue:focus-visible,
  .save:hover {
    color: #f3f3ff;
    background: transparent;
  }

  button:focus-visible,
  input:focus-visible,
  a:focus-visible {
    outline: 1px solid #f3f3ff;
    outline-offset: 2px;
  }

  @media (max-width: 640px) {
    .creator-layout {
      grid-template-columns: 1fr;
      gap: 16px;
    }

    .preview {
      min-height: 250px;
    }

    .avatar {
      width: min(28%, 120px);
    }

  }
</style>
