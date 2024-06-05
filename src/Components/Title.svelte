<section id="intro">
  <h1 id="intro-hed">Dietary Exploration</h1>
  <h2 class="intro-sub">Exploring the various facets of your diet and how it affects your health.</h2>
  <h2 id="intro__date">
  Dawson Tam, Paul Losiewicz, Nadine Marcus
  </h2>
</section>

<style>
  :root {
    --squid-ink: #333;
    --font-main: 'Arial, sans-serif';
    --background-color-main: #e7e0e8;
    --background-color-sub: #e6e6e6;
    --border-radius: 10px;
    --box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  }

  #intro {
    max-width: 800px;
    margin: 2rem auto;
    text-align: center;
    padding: 2rem;
    background-color: var(--background-color-main);
    border-radius: var(--border-radius);
    box-shadow: var(--box-shadow);
  }

  #intro-hed {
    font-size: 3rem;
    margin-top: 0;
    margin-bottom: 0.5rem;
    letter-spacing: 1.5px;
    text-transform: uppercase;
    color: var(--squid-ink);
    background-color: #e1cee1;
    padding: 1rem;
    border-radius: var(--border-radius);
  }

  .intro-sub {
    font-size: 1.5rem;
    color: var(--squid-ink);
    margin: 1rem 0;
    opacity: 0.85;
    padding: 1rem;
    background-color: #d7c8d7;
    border-radius: var(--border-radius);
    font-family: var(--font-main);
  }

  #intro__date {
    font-size: 2rem;
    color: var(--squid-ink);
    margin: 0;
    margin-top: 1rem;
    padding: 0.5rem;
    font-family: var(--font-main);
  }

  #intro__date a {
    color: var(--squid-ink);
    text-decoration: none;
  }

  #intro__date a:hover {
    text-decoration: underline;
  }

  /* mobile */
  @media screen and (max-width: 950px) {
    #intro-hed {
      font-size: 2.5rem;
    }

    .intro-sub {
      font-size: 1.2rem;
    }
  }
</style>
