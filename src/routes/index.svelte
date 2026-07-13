<script>
  import { onMount } from 'svelte';

  const KONAMI = [
    'ArrowUp',
    'ArrowUp',
    'ArrowDown',
    'ArrowDown',
    'ArrowLeft',
    'ArrowRight',
    'ArrowLeft',
    'ArrowRight',
    'KeyB',
    'KeyA'
  ];

  const LOADING_MESSAGES = [
    'Authenticating with SSO…',
    'Validating worktags…',
    'Locating your 47 nested approval chains…',
    'Calculating PTO balance (this may take 3–5 business days)…',
    'Syncing with the business process framework…',
    'Waiting for manager approval (manager status: on PTO since 2019)…',
    'Reconciling time blocks with fiscal calendar…',
    'Searching 12,847 custom reports for your pay stub…',
    'Applying security group inheritance…',
    'Running background process: Submit_Time_Off_v4.2.1…'
  ];

  const ERROR_MESSAGES = [
    'Your session has expired due to inactivity. Please sign in again.',
    'The task did not complete successfully. Contact your administrator.',
    'Error: Required worktag "Cost_Center" is missing.',
    'Business process "Request Time Off" is currently stuck at step 7 of 7.',
    'Report definition "My_Pay" not found. Did you mean "Someone_Elses_Pay"?',
    'You do not have sufficient access to view your own information.',
    'Workday is currently unavailable. Please try again during business hours (never).'
  ];

  let easterEggActive = false;
  let phase = 'loading'; // loading | error | done
  let loadingIndex = 0;
  let loadingProgress = 0;
  let errorIndex = 0;
  let typedBuffer = '';
  let konamiIndex = 0;
  let showHint = false;

  let loadingInterval;
  let errorInterval;

  function activateEasterEgg() {
    if (easterEggActive) return;
    easterEggActive = true;
    phase = 'loading';
    loadingIndex = 0;
    loadingProgress = 0;
    errorIndex = 0;
    startLoadingSequence();
  }

  function startLoadingSequence() {
    loadingInterval = setInterval(() => {
      loadingProgress += Math.random() * 3 + 0.5;
      if (loadingProgress > 100) loadingProgress = 100;

      if (Math.random() < 0.15 && loadingIndex < LOADING_MESSAGES.length - 1) {
        loadingIndex++;
      }

      if (loadingProgress >= 100) {
        clearInterval(loadingInterval);
        setTimeout(() => {
          phase = 'error';
          startErrorSequence();
        }, 800);
      }
    }, 200);
  }

  function startErrorSequence() {
    errorInterval = setInterval(() => {
      if (errorIndex < ERROR_MESSAGES.length - 1) {
        errorIndex++;
      } else {
        clearInterval(errorInterval);
        setTimeout(() => {
          phase = 'done';
        }, 1500);
      }
    }, 2200);
  }

  function closeEasterEgg() {
    easterEggActive = false;
    phase = 'loading';
    clearInterval(loadingInterval);
    clearInterval(errorInterval);
  }

  function handleKeydown(e) {
    if (e.key === 'Escape' && easterEggActive) {
      closeEasterEgg();
      return;
    }

    if (easterEggActive) return;

    // Konami code
    if (e.code === KONAMI[konamiIndex]) {
      konamiIndex++;
      if (konamiIndex === KONAMI.length) {
        konamiIndex = 0;
        activateEasterEgg();
      }
    } else {
      konamiIndex = 0;
    }

    // Type "workday"
    if (e.key.length === 1 && /[a-zA-Z]/.test(e.key)) {
      typedBuffer = (typedBuffer + e.key.toLowerCase()).slice(-7);
      if (typedBuffer.endsWith('workday')) {
        typedBuffer = '';
        activateEasterEgg();
      }
    }
  }

  onMount(() => {
    const hintTimer = setTimeout(() => {
      showHint = true;
    }, 8000);

    return () => {
      clearTimeout(hintTimer);
      clearInterval(loadingInterval);
      clearInterval(errorInterval);
    };
  });
</script>

<svelte:window on:keydown={handleKeydown} />

<marquee on:click={() => (showHint = true)}>Not Workday</marquee>

{#if showHint && !easterEggActive}
  <p class="hint">↑↑↓↓←→←→BA</p>
{/if}

{#if easterEggActive}
  <div class="overlay" role="dialog" aria-label="Workday experience simulator">
    <div class="workday-window">
      <header class="workday-header">
        <span class="workday-logo">workday</span>
        <span class="workday-user">Employee #4829103</span>
      </header>

      <div class="workday-body">
        {#if phase === 'loading'}
          <div class="loading-content">
            <div class="spinner" />
            <p class="loading-text">{LOADING_MESSAGES[loadingIndex]}</p>
            <div class="progress-bar">
              <div class="progress-fill" style="width: {loadingProgress}%" />
            </div>
            <p class="progress-label">{Math.floor(loadingProgress)}% complete</p>
          </div>
        {:else if phase === 'error'}
          <div class="error-content">
            <div class="error-icon">⚠</div>
            <h2>Something went wrong</h2>
            <div class="error-list">
              {#each ERROR_MESSAGES.slice(0, errorIndex + 1) as msg}
                <p class="error-msg">{msg}</p>
              {/each}
            </div>
          </div>
        {:else}
          <div class="done-content">
            <h2>Welcome to Workday!</h2>
            <p class="done-msg">
              You've successfully experienced the authentic Workday workflow:
              infinite loading, cryptic errors, and zero resolution.
            </p>
            <p class="done-sub">
              Your PTO request has been submitted and will be denied in 6–8 weeks.
            </p>
            <button class="escape-btn" on:click={closeEasterEgg}>
              Return to Not Workday
            </button>
            <p class="esc-hint">(or press Esc — unlike Workday, that actually works here)</p>
          </div>
        {/if}
      </div>
    </div>
  </div>
{/if}

<style>
  :global(body) {
    font-family: Menlo, Consolas, Monaco, Liberation Mono, Lucida Console,
      monospace;
    background-color: #fafafa;
    max-width: 650px;
    margin: 32px auto;
    padding: 0 16px;
  }

  h1 {
    letter-spacing: -0.025em;
  }

  .hint {
    text-align: center;
    color: #bbb;
    font-size: 11px;
    margin-top: 48px;
    user-select: none;
    letter-spacing: 0.1em;
  }

  .overlay {
    position: fixed;
    inset: 0;
    background: rgba(0, 0, 0, 0.6);
    display: flex;
    align-items: center;
    justify-content: center;
    z-index: 9999;
    padding: 16px;
  }

  .workday-window {
    background: #fff;
    border-radius: 4px;
    width: 100%;
    max-width: 520px;
    box-shadow: 0 8px 32px rgba(0, 0, 0, 0.3);
    overflow: hidden;
    font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif;
  }

  .workday-header {
    background: linear-gradient(135deg, #005cb9 0%, #0073e6 100%);
    color: #fff;
    padding: 12px 20px;
    display: flex;
    justify-content: space-between;
    align-items: center;
  }

  .workday-logo {
    font-size: 20px;
    font-weight: 300;
    letter-spacing: -0.5px;
    text-transform: lowercase;
  }

  .workday-user {
    font-size: 12px;
    opacity: 0.85;
  }

  .workday-body {
    padding: 32px 24px;
    min-height: 220px;
    display: flex;
    align-items: center;
    justify-content: center;
  }

  .loading-content {
    text-align: center;
    width: 100%;
  }

  .spinner {
    width: 48px;
    height: 48px;
    border: 4px solid #e0e0e0;
    border-top-color: #ff7a45;
    border-radius: 50%;
    margin: 0 auto 20px;
    animation: spin 1.2s linear infinite;
  }

  @keyframes spin {
    to {
      transform: rotate(360deg);
    }
  }

  .loading-text {
    color: #555;
    font-size: 14px;
    margin: 0 0 16px;
    min-height: 20px;
  }

  .progress-bar {
    background: #eee;
    border-radius: 4px;
    height: 8px;
    overflow: hidden;
    margin-bottom: 8px;
  }

  .progress-fill {
    background: #ff7a45;
    height: 100%;
    border-radius: 4px;
    transition: width 0.2s ease;
  }

  .progress-label {
    color: #999;
    font-size: 12px;
    margin: 0;
  }

  .error-content {
    text-align: center;
    width: 100%;
  }

  .error-icon {
    font-size: 40px;
    color: #ff7a45;
    margin-bottom: 8px;
  }

  .error-content h2 {
    color: #333;
    font-size: 18px;
    font-weight: 600;
    margin: 0 0 16px;
  }

  .error-list {
    text-align: left;
    max-height: 180px;
    overflow-y: auto;
  }

  .error-msg {
    background: #fff3f0;
    border-left: 3px solid #ff7a45;
    padding: 8px 12px;
    margin: 0 0 8px;
    font-size: 13px;
    color: #555;
    animation: fadeIn 0.3s ease;
  }

  @keyframes fadeIn {
    from {
      opacity: 0;
      transform: translateY(4px);
    }
    to {
      opacity: 1;
      transform: translateY(0);
    }
  }

  .done-content {
    text-align: center;
  }

  .done-content h2 {
    color: #005cb9;
    font-size: 20px;
    margin: 0 0 12px;
  }

  .done-msg {
    color: #555;
    font-size: 14px;
    line-height: 1.5;
    margin: 0 0 8px;
  }

  .done-sub {
    color: #888;
    font-size: 13px;
    font-style: italic;
    margin: 0 0 20px;
  }

  .escape-btn {
    background: #005cb9;
    color: #fff;
    border: none;
    padding: 10px 24px;
    border-radius: 4px;
    font-size: 14px;
    cursor: pointer;
    transition: background 0.2s;
  }

  .escape-btn:hover {
    background: #004a94;
  }

  .esc-hint {
    color: #aaa;
    font-size: 11px;
    margin: 12px 0 0;
  }
</style>
