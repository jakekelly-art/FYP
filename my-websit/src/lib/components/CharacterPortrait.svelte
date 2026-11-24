<script>
  import circleBody from '#lib/assets/CIRCLE_BODY.png';
  import circleFace from '#lib/assets/FACE_CIRCLE_1.png';

  let { label = 'Character preview' } = $props();
</script>

<div class="portrait" role="img" aria-label={label}>
  <img class="body" src={circleBody} alt="" />
  <img class="face" src={circleFace} alt="" />
</div>

<style>
  .portrait {
    position: relative;
    width: 100%;
    aspect-ratio: 1244 / 1534;
  }

  .body {
    display: block;
    width: 100%;
    height: 100%;
    object-fit: fill;
  }

  .face {
    position: absolute;
    top: 0;
    left: 16.5%;
    width: 67%;
    height: auto;
  }
</style>
