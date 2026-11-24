<script>
  import { fade } from 'svelte/transition';
</script>

<div class="home-page" in:fade={{ duration: 250 }}>
  <article class="home-panel">
    <h1>[WEBPET]</h1>
    <div class="panel-body">
      <p>
        Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nunc vitae sem vel lectus
        tincidunt consequat. Integer at libero non justo feugiat consequat. Aenean posuere,
        sapien sed vulputate tincidunt, arcu mauris malesuada erat, vitae tempus neque lorem
        in augue.
      </p>
      <p>
        Curabitur blandit tempus porttitor. Donec ullamcorper nulla non metus auctor
        fringilla. Vestibulum id ligula porta felis euismod semper. Maecenas faucibus
        mollis interdum, sed posuere consectetur est at lobortis.
      </p>
    </div>
  </article>
</div>

<style>
  .home-page {
    display: grid;
    width: 100%;
    min-height: 100%;
    padding: 48px clamp(24px, 6vw, 64px);
    place-items: center;
  }

  .home-panel {
    position: relative;
    width: min(700px, 100%);
    min-height: min(78vh, 630px);
    padding: clamp(72px, 12vw, 110px) clamp(24px, 5vw, 52px) 40px;
    border: 10px solid #fff;
    background: #fff;
    color: #1b1b1b;
  }

  h1 {
    position: absolute;
    top: 0;
    left: 50%;
    margin: 0;
    color: #1b1b1b;
    font-size: clamp(3rem, 8.5vw, 6.25rem);
    font-weight: 700;
    line-height: 0.9;
    white-space: nowrap;
    -webkit-text-stroke: 10px #fff;
    paint-order: stroke fill;
    transform: translate(-50%, -42%);
  }

  .panel-body {
    max-width: 560px;
    margin-inline: auto;
    font-size: clamp(1rem, 1.5vw, 1.2rem);
    line-height: 1.65;
  }

  .panel-body p {
    margin: 0 0 1.2em;
  }

  @media (max-width: 520px) {
    .home-page {
      padding: 36px 20px;
    }

    .home-panel {
      min-height: 72vh;
      padding-top: 68px;
    }
  }
</style>
