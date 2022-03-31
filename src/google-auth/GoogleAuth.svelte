<script>
  import { createEventDispatcher, onMount } from 'svelte'
  import loader from '@beyonk/async-script-loader'
  import { googleImage } from '../utils'

  const dispatch = createEventDispatcher()

  let signinCta
  export let clientId
  export let text = 'Sign in with Google'
  let disabled = true

  onMount(() => {
    loader(
      '//apis.google.com/js/api:client.js',
      () => window['gapi'],
      () => initialise()
    )
  })

  let GoogleAuth

  function initialise() {
    gapi.load('auth2', async () => {
      GoogleAuth = gapi.auth2.init({ client_id: clientId })
      GoogleAuth.then(attachHandler, handleInitialisationError)
    })
  }

  function attachHandler() {
    GoogleAuth.attachClickHandler(
      signinCta,
      {},
      () => dispatch('auth-success', { user: GoogleAuth.currentUser.get() }),
      (e) => dispatch('auth-failure', { error: e })
    )
    disabled = false
  }

  function handleInitialisationError(e) {
    console.error('gauth initialisation error', e)
    dispatch('init-error', { error: e })
  }
</script>

<div bind:this={signinCta} class="container">
  {#if $$slots?.default}
    <slot />
  {:else}
    <button {disabled} class="google-auth">
      <img src={googleImage} alt="Google" />
      <span>{text}</span>
    </button>
  {/if}
</div>

<style>
  .container {
    display: flex;
    height: auto;
    width: auto;
  }

  button.google-auth {
    width: 100%;
    border: 0;
    background-color: #4285f4;
    display: flex;
    justify-content: center;
    align-items: center;
    border-radius: 2px;
    padding: 5px 1px;
    cursor: pointer;
  }

  button.google-auth:disabled {
    background-color: grey;
  }

  .google-auth img {
    background: white;
    padding: 4px;
    height: 30px;
    width: 30px;
    border-radius: 2px;
    margin: 4px;
  }

  .google-auth span {
    font-family: Roboto, sans-serif;
    font-size: 14px;
    font-weight: bold;
    padding: 0 12px;
    color: white;
  }
</style>
