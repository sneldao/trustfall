<script>
  import ShowcaseScene from '$lib/ShowcaseScene.svelte';

  let activeExperiment = null;

  const experiments = [
    {
      name: 'latep',
      description: 'Play an XLM-staked dilemma without exposing your move.',
      tags: ['Stellar', 'ZK game'],
      href: 'https://latep.trustfall.xyz',
      status: 'Live'
    },
    {
      name: 'chime',
      description: 'Stake on crypto outcomes as AI agents debate both sides.',
      tags: ['AI agents', 'Prediction market'],
      href: 'https://chime.trustfall.xyz',
      status: 'Live'
    },
    {
      name: 'bothy',
      description: 'Accountable flood and winter-access decisions with human sign-off.',
      tags: ['Climate risk', 'Accountable AI'],
      href: 'https://bothyapp.netlify.app/',
      status: 'Live'
    },
    {
      name: 'elcaro',
      description: 'Detect indirect prompt injection before an agent can act on it.',
      tags: ['AI security', 'Threat detection'],
      href: 'https://elcaro.trustfall.xyz/',
      status: 'Live'
    },
    {
      name: 'srelok',
      description: 'Find and label unknown contracts for the Kleros Scout ecosystem.',
      tags: ['Kleros', 'Onchain curation'],
      href: 'https://srelok.netlify.app/',
      status: 'Live'
    },
    {
      name: 'claflin',
      description: 'A study of coordination is currently in research.',
      tags: ['In research'],
      status: 'Soon'
    }
  ];
</script>

<svelte:head>
  <title>trustfall — experiments in trust</title>
  <meta
    property="og:description"
    content="Small interactive experiments about trust, cooperation, and cryptography."
  />
</svelte:head>

<ShowcaseScene {activeExperiment} />

<main>
  <header>
    <a class="wordmark" href="/">trustfall</a>
    <p class="coordinates">studio / 2026</p>
  </header>

  <section class="intro" aria-labelledby="title">
    <p class="eyebrow">An independent field of play</p>
    <h1 id="title">Experiments<br />in trust.</h1>
    <p class="thesis">Games, systems, and strange proofs for learning how we cooperate.</p>
  </section>

  <section class="index" aria-labelledby="index-title">
    <div class="index-heading">
      <p id="index-title">Selected field notes</p>
      <p>01—0{experiments.length}</p>
    </div>

    <div class="experiments">
      {#each experiments as experiment, index}
        <article
          class:active={activeExperiment === index}
          class:inactive={!experiment.href}
          class="experiment"
          onmouseenter={() => (activeExperiment = index)}
          onmouseleave={() => (activeExperiment = null)}
        >
          <p class="number">0{index + 1}</p>
          <div class="project">
            <h2>{experiment.name}</h2>
            <p>{experiment.description}</p>
            <ul aria-label={`${experiment.name} categories`}>
              {#each experiment.tags as tag}
                <li>{tag}</li>
              {/each}
            </ul>
          </div>
          {#if experiment.href}
            <a
              class="visit"
              href={experiment.href}
              onfocus={() => (activeExperiment = index)}
              onblur={() => (activeExperiment = null)}
            >
              Visit <span aria-hidden="true">↗</span>
            </a>
          {:else}
            <span class="soon">{experiment.status}</span>
          {/if}
        </article>
      {/each}
    </div>
  </section>

  <footer>
    <span>© trustfall</span>
    <a href="https://github.com/thisyearnofear">GitHub ↗</a>
  </footer>
</main>

<style>
  :global(*) {
    box-sizing: border-box;
  }

  :global(html) {
    background: #090a0d;
  }

  :global(body) {
    margin: 0;
    background: #090a0d;
    color: #f1f2ed;
    font-family: 'DM Mono', monospace;
  }

  main {
    position: relative;
    z-index: 1;
    display: grid;
    grid-template-columns: minmax(1.5rem, 1fr) minmax(0, 72rem) minmax(1.5rem, 1fr);
    min-height: 100svh;
    pointer-events: none;
  }

  header,
  .intro,
  .index,
  footer {
    grid-column: 2;
    pointer-events: auto;
  }

  header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 1.5rem 0;
    border-bottom: 1px solid rgba(225, 255, 215, 0.2);
  }

  .wordmark {
    color: inherit;
    font-family: 'Instrument Serif', Georgia, serif;
    font-size: 2rem;
    line-height: 0.8;
    text-decoration: none;
  }

  .coordinates,
  .eyebrow,
  .index-heading,
  .number,
  .soon,
  footer {
    color: rgba(226, 255, 215, 0.82);
    font-size: 0.68rem;
    letter-spacing: 0.1em;
    text-transform: uppercase;
  }

  .intro {
    align-self: start;
    padding: clamp(5.5rem, 15vh, 10rem) 0 clamp(6rem, 18vh, 13rem);
    max-width: 47rem;
  }

  .eyebrow {
    margin: 0 0 1.25rem;
  }

  h1,
  h2,
  p {
    margin: 0;
  }

  h1 {
    font-family: 'Instrument Serif', Georgia, serif;
    font-size: clamp(5rem, 13vw, 10.5rem);
    font-weight: 400;
    letter-spacing: -0.07em;
    line-height: 0.73;
  }

  .thesis {
    max-width: 23rem;
    margin-top: 2rem;
    color: rgba(241, 242, 237, 0.78);
    font-size: 0.76rem;
    line-height: 1.65;
  }

  .index {
    padding-bottom: 4rem;
  }

  .index-heading {
    display: flex;
    justify-content: space-between;
    padding-bottom: 0.75rem;
    border-bottom: 1px solid rgba(225, 255, 215, 0.2);
  }

  .experiments {
    border-bottom: 1px solid rgba(225, 255, 215, 0.2);
  }

  .experiment {
    display: grid;
    grid-template-columns: 3.75rem minmax(0, 1fr) auto;
    gap: 1rem;
    align-items: center;
    min-height: 11.5rem;
    padding: 1.5rem 0;
    border-bottom: 1px solid rgba(225, 255, 215, 0.12);
  }

  .experiment:last-child {
    border-bottom: 0;
  }

  .experiment.active .project h2 {
    color: #d9ffca;
  }

  .project h2 {
    font-family: 'Instrument Serif', Georgia, serif;
    font-size: clamp(2.8rem, 5vw, 4.5rem);
    font-weight: 400;
    letter-spacing: -0.06em;
    line-height: 0.88;
  }

  .project > p {
    max-width: 25rem;
    margin-top: 0.75rem;
    color: rgba(241, 242, 237, 0.84);
    font-size: 0.75rem;
    line-height: 1.5;
  }

  ul {
    display: flex;
    flex-wrap: wrap;
    gap: 0.4rem;
    padding: 0;
    margin: 1rem 0 0;
    list-style: none;
  }

  li {
    padding: 0.28rem 0.45rem;
    border: 1px solid rgba(225, 255, 215, 0.26);
    color: rgba(226, 255, 215, 0.88);
    font-size: 0.62rem;
    letter-spacing: 0.06em;
    text-transform: uppercase;
  }

  .visit {
    padding: 0.7rem 0;
    color: #d9ffca;
    font-size: 0.68rem;
    letter-spacing: 0.04em;
    text-decoration: none;
    transition: transform 180ms ease, color 180ms ease;
  }

  .visit:hover,
  .visit:focus-visible {
    color: #fff;
    transform: translateX(0.25rem);
  }

  .inactive {
    opacity: 0.62;
  }

  footer {
    display: flex;
    justify-content: space-between;
    align-self: end;
    padding: 1.5rem 0;
  }

  footer a {
    color: inherit;
    text-decoration: none;
  }

  footer a:hover,
  footer a:focus-visible {
    color: #fff;
  }

  @media (max-width: 640px) {
    main {
      grid-template-columns: 1.25rem minmax(0, 1fr) 1.25rem;
    }

    header {
      padding-top: 1.25rem;
    }

    .coordinates {
      font-size: 0.52rem;
    }

    .intro {
      padding: 7.5rem 0 8rem;
    }

    h1 {
      font-size: clamp(4.8rem, 24vw, 7rem);
    }

    .experiment {
      grid-template-columns: 2rem minmax(0, 1fr);
      min-height: 0;
      padding: 1.5rem 0 1.25rem;
    }

    .visit,
    .soon {
      grid-column: 2;
      justify-self: start;
      margin-top: 0.25rem;
    }
  }
</style>
