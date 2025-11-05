<script>
	import { onMount } from 'svelte';
	import 'iconify-icon';

	function initializeCodeBlocks() {
		// Find all pre tags that haven't been processed yet
		const unprocessedBlocks = document.querySelectorAll('.prose-content pre:not([data-processed])');
		
		// Use DocumentFragment for batch DOM operations
		unprocessedBlocks.forEach(preEl => {
			preEl.setAttribute('data-processed', 'true');
			const codeEl = preEl.querySelector('code');
			if (!codeEl) return;

			// --- Line Numbers ---
			const codeText = codeEl.innerText;
			const lines = (codeText.endsWith('\n') ? codeText.slice(0, -1) : codeText).split('\n');
			const lineCount = lines.length;

			// Build line numbers HTML string for better performance
			const lineNumbersHTML = Array.from({ length: lineCount }, (_, i) => 
				`<div>${i + 1}</div>`
			).join('');

			const lineNumbersWrapper = document.createElement('div');
			lineNumbersWrapper.className = 'line-numbers-wrapper';
			lineNumbersWrapper.setAttribute('aria-hidden', 'true');
			lineNumbersWrapper.innerHTML = lineNumbersHTML;

			const container = document.createElement('div');
			container.className = 'code-container';
			container.appendChild(codeEl);
			container.insertBefore(lineNumbersWrapper, codeEl);
			preEl.appendChild(container);

			// --- Copy Button ---
			const copyButton = document.createElement('button');
			copyButton.className = 'copy-button';
			copyButton.title = 'Copy code';
			copyButton.innerHTML = `<iconify-icon icon="mdi:content-copy"></iconify-icon>`;

			copyButton.addEventListener('click', () => {
				navigator.clipboard.writeText(codeText).then(() => {
					copyButton.innerHTML = `<iconify-icon icon="mdi:check"></iconify-icon>`;
					copyButton.dataset.copied = 'true';
					setTimeout(() => {
						copyButton.innerHTML = `<iconify-icon icon="mdi:content-copy"></iconify-icon>`;
						delete copyButton.dataset.copied;
					}, 2000);
				});
			});
			preEl.insertBefore(copyButton, preEl.firstChild);
		});
	}

	// Initialize on page load
	onMount(() => {
		initializeCodeBlocks();
        
		document.addEventListener('astro:page-load', initializeCodeBlocks);
	});
</script>