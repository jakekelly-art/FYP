
<script>
  import { page } from '$app/state';
  import logo from '#lib/assets/LOGO.png';

  const links = [
    { name: 'Home', path: '/' },
    { name: 'Game', path: '/game' },
    { name: 'Photo Album', path: '/photo-album' },
    { name: 'Settings', path: '/settings' }
  ];
</script>

<aside>
  <div class="brand">
    <img src={logo} alt="Tomodachi Life" />
  </div>

  <nav>
    {#each links as link}
      <a
        href={link.path}
        class:active={page.url.pathname === link.path}
      >
        {link.name}
      </a>
    {/each}
  </nav>
</aside>

<style>
  aside {
    width: 200px;
    flex-shrink: 0;
    height: 100%;
    background: rgb(3 1 26 / 42%);
    -webkit-backdrop-filter: blur(18px);
    backdrop-filter: blur(18px);
    color: #f3f3ff;
    overflow-y: auto;
    border-right: 1px solid rgb(243 243 255 / 55%);
  }

  .brand {
    width: 100%;
  }

  .brand img {
    display: block;
    width: 100%;
    height: auto;
    object-fit: contain;
  }

  nav {
    display: flex;
    flex-direction: column;
    gap: 5px;
    padding: 18px 10px;
  }

  nav a {
    z-index: 0;
    padding: 12px 13px;
    border: 1px solid transparent;
    color: #f3f3ff;
    text-decoration: none;
    transition: transform 160ms ease;
  }

  nav a:hover,
  nav a:focus-visible {
    transform: translateX(8px);
  }

  nav a:focus-visible {
    outline: 1px solid #f3f3ff;
    outline-offset: 1px;
  }

  nav a.active {
    clip-path: polygon(0 0, 94% 0, 100% 26%, 100% 100%, 6% 100%, 0 74%);
    background: #f3f3ff;
    color: black;
  }

  @media (prefers-reduced-motion: reduce) {
    nav a {
      transition: none;
    }
  }
</style>
