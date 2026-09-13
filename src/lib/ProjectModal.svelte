<script>
  import { createEventDispatcher } from 'svelte';

  export let project;
  const dispatch = createEventDispatcher();
</script>

<svelte:window on:keydown={(event) => event.key === 'Escape' && dispatch('close')} />

<div class="backdrop" role="presentation" on:click={(event) => event.target === event.currentTarget && dispatch('close')}>
  <section class="modal" role="dialog" aria-modal="true" aria-labelledby="project-title">
    <button class="close" aria-label="Close project details" on:click={() => dispatch('close')}>×</button>

    <div class="media">
      {#if project.video}
        <video src={project.video} autoplay muted loop playsinline controls aria-label={`${project.name} project trailer`}></video>
      {:else}
        <div class="media-fallback" style={`--accent: ${project.palette[0]}; --accent-two: ${project.palette[1]}`}>
          <span>{project.kicker}</span>
          <strong>{project.name}</strong>
        </div>
      {/if}
    </div>

    <div class="details">
      <p class="eyebrow">{project.kicker}</p>
      <h2 id="project-title">{project.name}</h2>
      <p class="description">{project.details || project.description}</p>
      {#if project.highlights}
        <ul>
          {#each project.highlights as highlight}
            <li>{highlight}</li>
          {/each}
        </ul>
      {/if}
      <div class="actions">
        {#if project.href}
          <a href={project.href}>Visit project <span aria-hidden="true">↗</span></a>
        {/if}
        <button class="dismiss" on:click={() => dispatch('close')}>Back to field</button>
      </div>
    </div>
  </section>
</div>

<style>
  .backdrop {
    position: fixed;
    inset: 0;
    z-index: 10;
    display: grid;
    place-items: center;
    padding: 1rem;
    background: rgba(2, 2, 10, 0.68);
    backdrop-filter: blur(16px);
  }

  .modal {
    position: relative;
    width: min(760px, 100%);
    max-height: min(90svh, 760px);
    overflow: auto;
    border: 1px solid rgba(255, 255, 255, 0.24);
    border-radius: 1rem;
    background: #0c0c18;
    box-shadow: 0 2rem 8rem rgba(0, 0, 0, 0.55);
  }

  .close {
    position: absolute;
    top: 0.8rem;
    right: 0.8rem;
    z-index: 2;
    width: 2.4rem;
    height: 2.4rem;
    border: 1px solid rgba(255, 255, 255, 0.4);
    border-radius: 50%;
    color: white;
    background: rgba(0, 0, 0, 0.3);
    font-size: 1.5rem;
    cursor: pointer;
  }

  .media,
  .media video,
  .media-fallback {
    width: 100%;
    aspect-ratio: 16 / 9;
  }

  .media {
    overflow: hidden;
    background: #17172a;
  }

  .media video {
    display: block;
    object-fit: cover;
  }

  .media-fallback {
    display: grid;
    place-content: center;
    gap: 0.7rem;
    padding: 2rem;
    background: radial-gradient(circle at 25% 20%, var(--accent), transparent 42%), linear-gradient(135deg, var(--accent-two), #101020);
  }

  .media-fallback span,
  .eyebrow,
  li,
  .dismiss {
    color: rgba(255, 255, 255, 0.66);
    font: 0.64rem/1.3 'DM Mono', monospace;
    letter-spacing: 0.1em;
    text-transform: uppercase;
  }

  .media-fallback strong {
    font: 400 clamp(4rem, 12vw, 8rem)/0.8 'Instrument Serif', Georgia, serif;
  }

  .details {
    padding: clamp(1.2rem, 4vw, 2.5rem);
  }

  .eyebrow {
    margin: 0;
  }

  h2 {
    margin: 0.6rem 0 1rem;
    font: 400 clamp(3rem, 8vw, 6rem)/0.8 'Instrument Serif', Georgia, serif;
    letter-spacing: -0.06em;
  }

  .description {
    max-width: 38rem;
    margin: 0;
    color: rgba(255, 255, 255, 0.82);
    font-size: 0.8rem;
    line-height: 1.65;
  }

  ul {
    display: flex;
    flex-wrap: wrap;
    gap: 0.45rem;
    padding: 0;
    margin: 1.4rem 0 0;
    list-style: none;
  }

  li {
    padding: 0.4rem 0.55rem;
    border: 1px solid rgba(255, 255, 255, 0.22);
    color: rgba(255, 255, 255, 0.82);
  }

  .actions {
    display: flex;
    flex-wrap: wrap;
    gap: 0.7rem;
    align-items: center;
    margin-top: 1.7rem;
  }

  .actions a,
  .dismiss {
    padding: 0.75rem 0.9rem;
    border: 1px solid rgba(255, 255, 255, 0.3);
    color: white;
    background: transparent;
    text-decoration: none;
    cursor: pointer;
  }

  .actions a {
    border-color: #d9ffca;
    color: #d9ffca;
  }
</style>
