<script>
  import { onMount } from 'svelte';
  import circleBodySource from '#lib/assets/CIRCLE_BODY.png';
  import circleFaceSource from '#lib/assets/FACE_CIRCLE_1.png';

  let { showCharacter = true } = $props();

  /** @type {HTMLCanvasElement} */
  let canvas;

  onMount(() => {
    const context = canvas.getContext('2d');
    if (!context) {
      throw new Error('Unable to create the game room canvas context.');
    }
    const reducedMotion = window.matchMedia('(prefers-reduced-motion: reduce)').matches;
    const circleBody = new Image();
    const circleFace = new Image();
    localStorage.removeItem('pet-room-hunger-v1');

    /**
     * @param {Array<[number, number]>} points
     * @param {string} color
     */
    const drawSurface = (points, color) => {
      context.beginPath();
      context.moveTo(points[0][0], points[0][1]);
      for (const [x, y] of points.slice(1)) {
        context.lineTo(x, y);
      }
      context.closePath();
      context.fillStyle = color;
      context.fill();
    };

    const playerHeight = 180;
    const playerWidth = playerHeight * (768 / 1110);
    const storageKey = 'pet-room-movement-v1';
    const movement = {
      x: 400,
      y: 640,
      startX: 400,
      startY: 640,
      targetX: 400,
      targetY: 640,
      walkStartedAt: 0,
      walkDuration: 0,
      pauseUntil: Date.now() + 1400,
      pauseDuration: 0,
      walking: false,
      savedAt: Date.now()
    };
    const idleAction = {
      type: 'none',
      startedAt: Date.now(),
      duration: 0,
      direction: 1
    };
    const gaze = {
      pointerX: 400,
      inside: false
    };
    let nextIdleAt = Date.now() + 1800 + Math.random() * 2600;
    let nextLookAt = Date.now() + 9000 + Math.random() * 9000;

    const savedState = localStorage.getItem(storageKey);
    if (savedState) {
      const parsed = JSON.parse(savedState);
      const numericFields = [
        'x',
        'y',
        'startX',
        'startY',
        'targetX',
        'targetY',
        'walkStartedAt',
        'walkDuration',
        'pauseUntil',
        'pauseDuration',
        'savedAt'
      ];

      if (
        typeof parsed !== 'object' ||
        parsed === null ||
        typeof parsed.walking !== 'boolean' ||
        numericFields.some((field) => !Number.isFinite(parsed[field]))
      ) {
        throw new Error('Saved pet movement data is invalid.');
      }

      Object.assign(movement, parsed);
    }

    const drawRoom = () => {
      context.clearRect(0, 0, canvas.width, canvas.height);
      drawSurface(
        [
          [0, 0],
          [400, 0],
          [400, 376],
          [0, 564]
        ],
        '#101010'
      );
      drawSurface(
        [
          [400, 0],
          [800, 0],
          [800, 564],
          [400, 376]
        ],
        '#ff0000'
      );
      drawSurface(
        [
          [0, 564],
          [400, 376],
          [800, 564],
          [400, 800]
        ],
        '#e8e8e8'
      );
    };

    /** @param {number} y */
    const getFloorBounds = (y) => {
      if (y <= 564) {
        const depth = (y - 376) / (564 - 376);
        return {
          left: 400 * (1 - depth),
          right: 400 * (1 + depth)
        };
      }

      const depth = (y - 564) / (800 - 564);
      return {
        left: 400 * depth,
        right: 800 - 400 * depth
      };
    };

    /** @param {number} time */
    const chooseDestination = (time) => {
      movement.startX = movement.x;
      movement.startY = movement.y;

      const targetY = 440 + Math.random() * 290;
      const scaleAtDepth = 0.62 + ((targetY - 440) / 290) * 0.38;
      const bounds = getFloorBounds(targetY);
      const horizontalPadding = (playerWidth * scaleAtDepth) / 2 + 14;
      const minX = bounds.left + horizontalPadding;
      const maxX = bounds.right - horizontalPadding;
      const targetX = minX + Math.random() * Math.max(0, maxX - minX);
      const distance = Math.hypot(targetX - movement.x, targetY - movement.y);

      movement.targetX = targetX;
      movement.targetY = targetY;
      movement.walkStartedAt = time;
      movement.walkDuration = Math.max(1800, distance * 10);
      movement.pauseDuration = 1200 + Math.random() * 1800;
      movement.walking = true;
    };

    /** @param {number} time */
    const updateMovement = (time) => {
      if (reducedMotion) return;

      if (!movement.walking) {
        if (idleAction.type !== 'none' && time >= idleAction.startedAt + idleAction.duration) {
          idleAction.type = 'none';
        }

        if (
          idleAction.type === 'none' &&
          gaze.inside &&
          time >= movement.pauseUntil &&
          time >= nextLookAt
        ) {
          if (Math.random() < 0.08) {
            idleAction.type = 'look';
            idleAction.startedAt = time;
            idleAction.duration = 1800 + Math.random() * 1600;
            idleAction.direction = gaze.pointerX < movement.x ? -1 : 1;
          }
          nextLookAt = time + 8000 + Math.random() * 10000;
        }

        if (time >= movement.pauseUntil && idleAction.type === 'none') {
          chooseDestination(time);
          return;
        }

        if (time >= nextIdleAt) {
          const actions = ['stretch', 'lean', 'hop'];
          idleAction.type = actions[Math.floor(Math.random() * actions.length)];
          idleAction.startedAt = time;
          idleAction.duration = 700 + Math.random() * 600;
          idleAction.direction = Math.random() < 0.5 ? -1 : 1;
          nextIdleAt = time + idleAction.duration + 1800 + Math.random() * 3200;
        }
        return;
      }

      idleAction.type = 'none';
      const progress = Math.min(1, (time - movement.walkStartedAt) / movement.walkDuration);
      const easedProgress = progress * progress * (3 - 2 * progress);
      movement.x = movement.startX + (movement.targetX - movement.startX) * easedProgress;
      movement.y = movement.startY + (movement.targetY - movement.startY) * easedProgress;

      if (progress === 1) {
        movement.walking = false;
        movement.pauseUntil = movement.walkStartedAt + movement.walkDuration + movement.pauseDuration;
      }
    };

    const drawPlayer = (time = 0) => {
      const idlePhase = time / 1000;
      const idleProgress = idleAction.duration
        ? Math.max(0, Math.min(1, (time - idleAction.startedAt) / idleAction.duration))
        : 0;
      const actionStrength = Math.sin(idleProgress * Math.PI);
      let bob = 0;
      let scaleX = 1;
      let scaleY = 1;
      let rotation = 0;

      if (!reducedMotion && movement.walking) {
        const step = Math.abs(Math.sin((time - movement.walkStartedAt) / 150));
        bob = -step * 4;
        scaleY = 1 - step * 0.018;
        scaleX = 1 + step * 0.01;
      } else if (!reducedMotion) {
        bob = Math.sin(idlePhase * 2) * 1.5;
        scaleY = 1 + Math.sin(idlePhase * 1.4) * 0.008;
        if (idleAction.type === 'stretch') {
          scaleY += actionStrength * 0.055;
          scaleX -= actionStrength * 0.025;
          bob -= actionStrength * 3;
        } else if (idleAction.type === 'pet') {
          const petPose = interpolatePetPose(idleProgress);
          scaleY *= petPose.scaleY;
          scaleX *= petPose.scaleX;
          bob += petPose.bob;
          rotation = petPose.lean * idleAction.direction;
        } else if (idleAction.type === 'look') {
          const currentDirection = gaze.pointerX < movement.x ? -1 : 1;
          rotation = actionStrength * 0.045 * currentDirection;
          bob -= actionStrength * 1.5;
        } else if (idleAction.type === 'lean') {
          rotation = actionStrength * 0.07 * idleAction.direction;
        } else if (idleAction.type === 'hop') {
          bob -= actionStrength * 12;
          scaleY -= actionStrength * 0.035;
          scaleX += actionStrength * 0.025;
        }
      }

      const depthScale = 0.62 + ((movement.y - 440) / 290) * 0.38;

      context.save();
      context.translate(movement.x, movement.y + bob);
      context.rotate(rotation);
      context.scale(depthScale * scaleX, depthScale * scaleY);
      context.fillStyle = 'rgb(0 133 255 / 28%)';
      context.fillRect(-playerWidth / 2, -playerHeight, playerWidth, playerHeight);
      const spriteHeight = circleBody.naturalWidth
        ? playerWidth * (circleBody.naturalHeight / circleBody.naturalWidth)
        : playerWidth * (1534 / 1244);
      if (circleBody.complete && circleBody.naturalWidth > 0) {
        context.drawImage(
          circleBody,
          -playerWidth / 2,
          -spriteHeight,
          playerWidth,
          spriteHeight
        );
      }
      if (circleFace.complete && circleFace.naturalWidth > 0) {
        const faceSize = playerWidth * 0.65;
        const faceHeight = faceSize * (circleFace.naturalHeight / circleFace.naturalWidth);
        const faceX = -faceSize / 2;
        const faceY = -spriteHeight + playerWidth * 0.09;
        context.drawImage(circleFace, faceX, faceY, faceSize, faceHeight);
      }
      context.restore();
    };

    /**
     * @param {number} progress
     * @returns {{ scaleX: number, scaleY: number, bob: number, lean: number }}
     */
    const interpolatePetPose = (progress) => {
      const poses = [
        { at: 0, scaleX: 1, scaleY: 1, bob: 0, lean: 0 },
        { at: 0.18, scaleX: 1.08, scaleY: 0.84, bob: 2, lean: 0.025 },
        { at: 0.48, scaleX: 0.93, scaleY: 1.16, bob: -4, lean: 0.09 },
        { at: 0.7, scaleX: 1.03, scaleY: 0.97, bob: 1, lean: 0.035 },
        { at: 1, scaleX: 1, scaleY: 1, bob: 0, lean: 0 }
      ];

      for (let index = 1; index < poses.length; index += 1) {
        const nextPose = poses[index];
        if (progress <= nextPose.at) {
          const previousPose = poses[index - 1];
          const segmentProgress =
            (progress - previousPose.at) / (nextPose.at - previousPose.at);
          const easedProgress =
            segmentProgress * segmentProgress * (3 - 2 * segmentProgress);

          return {
            scaleX: previousPose.scaleX + (nextPose.scaleX - previousPose.scaleX) * easedProgress,
            scaleY: previousPose.scaleY + (nextPose.scaleY - previousPose.scaleY) * easedProgress,
            bob: previousPose.bob + (nextPose.bob - previousPose.bob) * easedProgress,
            lean: previousPose.lean + (nextPose.lean - previousPose.lean) * easedProgress
          };
        }
      }

      return { scaleX: 1, scaleY: 1, bob: 0, lean: 0 };
    };

    const isPetIdle = () =>
      !movement.walking && idleAction.type === 'none' && Date.now() < movement.pauseUntil;

    /**
     * @param {number} time
     * @param {number} pointerX
     */
    const startPetting = (time, pointerX) => {
      if (!isPetIdle()) return;

      idleAction.type = 'pet';
      idleAction.startedAt = time;
      idleAction.duration = 1100;
      idleAction.direction = pointerX < movement.x ? -1 : 1;
      nextIdleAt = time + idleAction.duration + 1200;
    };

    /** @param {PointerEvent} event */
    const handlePointerMove = (event) => {
      const bounds = canvas.getBoundingClientRect();
      const scaleX = 800 / bounds.width;
      const scaleY = 800 / bounds.height;
      const pointerX = (event.clientX - bounds.left) * scaleX;
      const pointerY = (event.clientY - bounds.top) * scaleY;
      gaze.pointerX = pointerX;
      gaze.inside =
        pointerX >= 0 && pointerX <= 800 && pointerY >= 0 && pointerY <= 800;

      const depthScale = 0.62 + ((movement.y - 440) / 290) * 0.38;
      const hitWidth = (playerWidth * depthScale) / 2;
      const hitHeight = playerHeight * depthScale;
      const isOverPlayer =
        pointerX >= movement.x - hitWidth &&
        pointerX <= movement.x + hitWidth &&
        pointerY >= movement.y - hitHeight &&
        pointerY <= movement.y;

      canvas.style.cursor = isOverPlayer && isPetIdle() ? 'pointer' : 'default';
    };

    /** @param {PointerEvent} event */
    const handlePointerDown = (event) => {
      const bounds = canvas.getBoundingClientRect();
      const pointerX = (event.clientX - bounds.left) * (800 / bounds.width);
      const pointerY = (event.clientY - bounds.top) * (800 / bounds.height);
      const depthScale = 0.62 + ((movement.y - 440) / 290) * 0.38;
      const hitWidth = (playerWidth * depthScale) / 2;
      const hitHeight = playerHeight * depthScale;
      const hitPlayer =
        pointerX >= movement.x - hitWidth &&
        pointerX <= movement.x + hitWidth &&
        pointerY >= movement.y - hitHeight &&
        pointerY <= movement.y;

      if (idleAction.type === 'look') idleAction.type = 'none';
      if (hitPlayer) startPetting(Date.now(), pointerX);
    };

    const handlePointerLeave = () => {
      canvas.style.cursor = 'default';
      gaze.inside = false;
      if (idleAction.type === 'look') idleAction.type = 'none';
    };

    /** @param {KeyboardEvent} event */
    const handleKeyDown = (event) => {
      if (event.key === 'Enter' || event.key === ' ') {
        event.preventDefault();
        startPetting(Date.now(), movement.x);
      }
    };

    const drawFrame = (time = Date.now()) => {
      updateMovement(time);
      drawRoom();
      if (showCharacter) drawPlayer(time);
    };

    circleBody.onload = () => drawFrame();
    circleBody.onerror = () => {
      throw new Error(`Unable to load player body sprite: ${circleBodySource}`);
    };
    circleBody.src = circleBodySource;
    circleFace.onload = () => drawFrame();
    circleFace.onerror = () => {
      throw new Error(`Unable to load player face sprite: ${circleFaceSource}`);
    };
    circleFace.src = circleFaceSource;

    const persistMovement = () => {
      movement.savedAt = Date.now();
      localStorage.setItem(
        storageKey,
        JSON.stringify({
          ...movement
        })
      );
    };

    const now = Date.now();
    if (now - movement.savedAt > 10 * 60 * 1000) {
      chooseDestination(now);
    }

    drawFrame(now);
    let animationFrame = 0;
    let saveInterval = 0;

    const handlePageHide = () => {
      persistMovement();
    };
    window.addEventListener('pagehide', handlePageHide);
    window.addEventListener('pointermove', handlePointerMove);
    canvas.addEventListener('pointerleave', handlePointerLeave);
    canvas.addEventListener('pointerdown', handlePointerDown);
    canvas.addEventListener('keydown', handleKeyDown);

    const animate = () => {
      drawFrame(Date.now());
      if (!reducedMotion) {
        animationFrame = window.requestAnimationFrame(animate);
      } else {
        animationFrame = 0;
      }
    };
    const startAnimation = () => {
      if (!animationFrame) animationFrame = window.requestAnimationFrame(animate);
    };
    if (!reducedMotion) startAnimation();
    saveInterval = window.setInterval(() => {
      persistMovement();
      drawFrame(Date.now());
    }, 2000);

    return () => {
      window.cancelAnimationFrame(animationFrame);
      window.clearInterval(saveInterval);
      window.removeEventListener('pagehide', handlePageHide);
      window.removeEventListener('pointermove', handlePointerMove);
      canvas.removeEventListener('pointerleave', handlePointerLeave);
      canvas.removeEventListener('pointerdown', handlePointerDown);
      canvas.removeEventListener('keydown', handleKeyDown);
      persistMovement();
    };
  });
</script>

<div class="game-room">
  <canvas
    bind:this={canvas}
    width="800"
    height="800"
    tabindex="0"
    role="button"
    aria-label="Pet room. Click the pet while idle to pet it."
  ></canvas>
</div>

<style>
  .game-room {
    position: relative;
    width: min(800px, 100%);
    margin-inline: auto;
  }

  canvas {
    display: block;
    width: 100%;
    height: auto;
    aspect-ratio: 1;
    background: transparent;
    touch-action: none;
    user-select: none;
  }

</style>
