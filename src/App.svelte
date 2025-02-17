<script>
  import { onMount } from 'svelte';
  import { writable, derived } from 'svelte/store';

  let websiteUrl = '';
  let appleStoreUrl = '';
  let androidStoreUrl = '';
  let generatedUrl = '';
  const error = writable('');
  let isLoading = false;
  let copyMessage = null;

  const showError = derived(error, ($error) => $error !== '');

  const isValidUrl = (url) => {
    try {
      new URL(url);
      return true;
    } catch (_) {
      return false;
    }
  };

  async function copyToClipboard() {
    await navigator.clipboard.writeText(generatedUrl);
    copyMessage = 'Copied to clipboard!';
    setTimeout(() => {
      copyMessage = null;
    }, 2000);
  }

  

  async function handleSubmit(event) {
    event.preventDefault();
    error.set('');
    isLoading = true;

    if (!isValidUrl(websiteUrl) || !isValidUrl(appleStoreUrl) || !isValidUrl(androidStoreUrl)) {
      error.set('Please provide valid URLs for all fields.');
      isLoading = false;
      return;
    }

    const payload = { websiteUrl, appleStoreUrl, androidStoreUrl };
    const response = await fetch('https://be-linkergen.onrender.com/api/generate', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(payload),
    });

    if (response.ok) {
      const data = await response.json();
      generatedUrl = data.generatedUrl;
      console.log('Generated URL:', generatedUrl);
      clearForm();
    } else {
      error.set('Error generating URL. Please try again.');
    }

    isLoading = false;
  }

  function clearForm() {
    websiteUrl = '';
    appleStoreUrl = '';
    androidStoreUrl = '';
  }
</script>

<style>

  /* Google-inspired Material Design styles */
  body {
    font-family: 'Roboto', sans-serif;
  }

  form {
    background-color: white;
    max-width: 500px;
    margin: 40px auto;
    padding: 32px;
    border-radius: 8px;
    box-shadow: 0 1px 3px rgba(0, 0, 0, 0.12), 0 1px 2px rgba(0, 0, 0, 0.24);
  }

  label {
    display: block;
    font-size: 16px;
    font-weight: 500;
    margin-bottom: 8px;
  }

  input[type='url'] {
    width: 100%;
    font-size: 16px;
    padding: 8px;
    margin-bottom: 16px;
    border: none;
    border-bottom: 2px solid #4a4a4a;
    outline: none;
    transition: border-color 0.3s;
  }

  input[type='url']:focus {
    border-bottom-color: #1a73e8;
  }

  button {
    background-color: #1a73e8;
    color: white;
    border: none;
    border-radius: 4px;
    padding: 8px 16px;
    font-size: 16px;
    font-weight: 500;
    cursor: pointer;
    transition: background-color 0.3s;
  }

  button:hover {
    background-color: #1565c0;
  }

  p {
    color: #d32f2f;
    font-weight: 500;
  }

  .copy-message {
  display: inline-block;
  background-color: #4caf50;
  color: white;
  padding: 6px 12px;
  border-radius: 4px;
  font-size: 14px;
  margin-left: 8px;
  transition: all 0.3s;
}
 header {
    text-align: center;
    margin-bottom: 24px;
  }

  h1 {
    font-size: 36px;
    margin-bottom: 16px;
  }

  p {
    font-size: 18px;
    line-height: 1.5;
    max-width: 800px;
    margin: 0 auto;
  }
</style>
<header>
  <h1>LinkerGen</h1>
  <p>
    LinkerGen is a powerful tool that simplifies sharing links across different platforms. It generates a single universal link that automatically redirects users to the appropriate app store or website, depending on their device and operating system. This tool is particularly useful for marketers, developers, and content creators who need to share their apps or websites with a diverse audience.
  </p>
</header>
<form on:submit={handleSubmit}>
  <div>
    <label for="websiteUrl">Website URL:</label>
    <input type="url" id="websiteUrl" bind:value={websiteUrl} required />
  </div>
  <div>
    <label for="appleStoreUrl">Apple Store URL:</label>
    <input type="url" id="appleStoreUrl" bind:value={appleStoreUrl} required />
  </div>
  <div>
    <label for="androidStoreUrl">Android Store URL:</label>
    <input type="url" id="androidStoreUrl" bind:value={androidStoreUrl} required />
  </div>
 <button
    type="submit"
    class="generate-button"
    disabled={isLoading}
  >
    {isLoading ? 'Generating...' : 'Generate'}
  </button>
  
  {#if $showError}
    <p>{$error}</p>
  {/if}
</form>

{#if generatedUrl}
  <div>
    <span>Generated URL:</span>
    <a href="{generatedUrl}" target="_blank" rel="noopener noreferrer">
      {generatedUrl}
    </a>
    <button on:click={copyToClipboard}>
      Copy
    </button>
  </div>
{/if}

{#if copyMessage}
  <div class="copy-message">
    {copyMessage}
  </div>
{/if}
