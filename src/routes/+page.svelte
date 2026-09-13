<script>
  import ShowcaseScene from '$lib/ShowcaseScene.svelte';

  const projects = [
    {
      name: 'latep',
      kicker: 'ZK game · Stellar',
      description: 'Trust, sealed before it is revealed.',
      href: 'https://latep.trustfall.xyz',
      palette: ['#ff5c35', '#f5d67b', '#35111d']
    },
    {
      name: 'chime',
      kicker: 'AI agents · Markets',
      description: 'Follow or fade the window.',
      href: 'https://chime.trustfall.xyz',
      palette: ['#ffdc2e', '#ff6247', '#351163']
    },
    {
      name: 'bothy',
      kicker: 'Climate risk · Decisions',
      description: 'The agent watches. The human calls.',
      href: 'https://bothyapp.netlify.app/',
      palette: ['#d8ff70', '#5d8e73', '#172a34']
    },
    {
      name: 'elcaro',
      kicker: 'AI security · Detection',
      description: 'See what your agent cannot.',
      href: 'https://elcaro.trustfall.xyz/',
      palette: ['#78f7ff', '#2365e8', '#08152d']
    },
    {
      name: 'srelok',
      kicker: 'Kleros · Curation',
      description: 'Names for the untagged.',
      href: 'https://srelok.netlify.app/',
      palette: ['#ff4f91', '#9339ff', '#1b0c31']
    },
    {
      name: 'claflin',
      kicker: 'Research · Soon',
      description: 'A study of coordination.',
      palette: ['#dad8cf', '#777d82', '#181b1e']
    }
  ];

  let activeProject = null;
</script>

<svelte:head>
  <title>trustfall — experiments in trust</title>
  <meta
    name="description"
    content="An infinite field of experiments in trust, cooperation, AI, and cryptography."
  />
</svelte:head>

<ShowcaseScene
  {projects}
  on:projectfocus={(event) => (activeProject = event.detail)}
  on:projectblur={() => (activeProject = null)}
/>

<header class="chrome">
  <a class="wordmark" href="/" aria-label="trustfall home">trustfall</a>
  <p>experiments in trust</p>
</header>

<div class="instructions" aria-hidden="true">
  <span class="pulse"></span>
  drag to explore
</div>

<div class="project-status" aria-live="polite">
  {#if activeProject}
    <span>{activeProject.kicker}</span>
    <strong>{activeProject.name}</strong>
  {:else}
    <span>six experiments</span>
    <strong>move through the field</strong>
  {/if}
</div>

<footer>
  <p>Independent systems, games &amp; strange proofs.</p>
  <a href="https://github.com/sneldao/trustfall">Source ↗</a>
</footer>

<nav class="accessible-index" aria-label="Projects">
  {#each projects as project}
    {#if project.href}
      <a href={project.href}>{project.name}: {project.description}</a>
    {:else}
      <span>{project.name}: {project.description} Coming soon.</span>
    {/if}
  {/each}
</nav>

<style>
  :global(*) {
    box-sizing: border-box;
  }

  :global(html) {
    background: #03030b;
    color-scheme: dark;
  }

  :global(body) {
    margin: 0;
    overflow: hidden;
    background: #03030b;
    color: #f8f8f3;
    font-family: 'DM Mono', monospace;
  }

  .chrome,
  footer,
  .instructions,
  .project-status {
    position: fixed;
    z-index: 3;
    pointer-events: none;
  }

  .chrome {
    top: 0;
    left: 0;
    display: flex;
    width: 100%;
    align-items: baseline;
    justify-content: space-between;
    padding: 1.25rem 1.4rem;
  }

  .wordmark,
  footer a {
    pointer-events: auto;
  }

  .wordmark {
    color: inherit;
    font-family: 'Instrument Serif', Georgia, serif;
    font-size: clamp(2rem, 3vw, 3rem);
    line-height: 0.8;
    text-decoration: none;
  }

  .chrome p,
  footer,
  .instructions,
  .project-status span {
    margin: 0;
    color: rgba(255, 255, 255, 0.72);
    font-size: 0.64rem;
    letter-spacing: 0.11em;
    text-transform: uppercase;
  }

  .instructions {
    top: 50%;
    left: 1.4rem;
    display: flex;
    align-items: center;
    gap: 0.55rem;
    transform: translateY(-50%) rotate(-90deg) translateX(-50%);
    transform-origin: left top;
  }

  .pulse {
    width: 0.42rem;
    height: 0.42rem;
    border: 1px solid currentColor;
    border-radius: 50%;
    animation: pulse 1.8s ease-out infinite;
  }

  .project-status {
    left: 50%;
    bottom: 1.25rem;
    display: grid;
    justify-items: center;
    gap: 0.25rem;
    transform: translateX(-50%);
    text-align: center;
  }

  .project-status strong {
    font-family: 'Instrument Serif', Georgia, serif;
    font-size: clamp(1.25rem, 2vw, 2rem);
    font-weight: 400;
    line-height: 1;
  }

  footer {
    right: 1.4rem;
    bottom: 1.25rem;
    left: 1.4rem;
    display: flex;
    align-items: center;
    justify-content: space-between;
  }

  footer a {
    color: inherit;
    text-decoration: none;
  }

  .accessible-index {
    position: fixed;
    width: 1px;
    height: 1px;
    overflow: hidden;
    clip-path: inset(50%);
    white-space: nowrap;
  }

  .accessible-index:focus-within {
    z-index: 10;
    top: 1rem;
    left: 1rem;
    display: grid;
    width: min(24rem, calc(100vw - 2rem));
    height: auto;
    gap: 0.5rem;
    padding: 1rem;
    overflow: visible;
    clip-path: none;
    background: #080812;
    white-space: normal;
  }

  .accessible-index a,
  .accessible-index span {
    color: white;
  }

  @keyframes pulse {
    0% {
      box-shadow: 0 0 0 0 rgba(255, 255, 255, 0.45);
    }
    80%,
    100% {
      box-shadow: 0 0 0 0.65rem rgba(255, 255, 255, 0);
    }
  }

  @media (max-width: 640px) {
    .chrome {
      padding: 1rem;
    }

    .chrome p,
    .instructions,
    footer p {
      display: none;
    }

    footer {
      right: 1rem;
      bottom: 1rem;
      left: auto;
    }

    .project-status {
      bottom: 1rem;
      max-width: 70vw;
    }
  }

  @media (prefers-reduced-motion: reduce) {
    .pulse {
      animation: none;
    }
  }
</style>
