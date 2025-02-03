<!-- ImageScanner.svelte -->
<script lang="ts">
	import Tesseract from 'tesseract.js';

	// Rune states
	let imageUrl = $state('');
	let isDragging = $state(false);
	let isProcessing = $state(false);
	let extractedText = $state('');
	let progress = $state(0);
	let error = $state('');

	const acceptedTypes = ['image/png', 'image/jpeg', 'image/webp'];

	// Handle drag events
	function handleDragOver(e: DragEvent) {
		e.preventDefault();
		isDragging = true;
	}

	function handleDragLeave(e: DragEvent) {
		e.preventDefault();
		isDragging = false;
	}

	async function handleDrop(e: DragEvent) {
		e.preventDefault();
		isDragging = false;

		const file = e.dataTransfer.files[0];
		if (!file || !acceptedTypes.includes(file.type)) {
			error = 'Invalid file type. Please upload an image.';
			return;
		}

		processImage(file);
	}

	// File input fallback
	async function handleFileInput(e: DragEvent) {
		const file = e.target.files[0];
		if (file) processImage(file);
	}

	async function processImage(file) {
		try {
			isProcessing = true;
			error = '';
			extractedText = '';

			// Step 1: Read image
			imageUrl = URL.createObjectURL(file);

			// Step 2: Detect document boundaries
			const img = await loadImage(imageUrl);

			// Step 3: Perform OCR
			const {
				data: { text }
			} = await Tesseract.recognize(img, 'eng', {
				logger: (m) => {
					if (m.status === 'recognizing text') {
						progress = Math.round(m.progress * 100);
					}
				}
			});

			extractedText = text.trim();
		} catch (err) {
			error = `Error processing image: ${err.message}`;
		} finally {
			isProcessing = false;
			progress = 0;
		}
	}

	function loadImage(src: string) {
		return new Promise((resolve, reject) => {
			const img = new Image();
			img.onload = () => resolve(img);
			img.onerror = reject;
			img.src = src;
		});
	}
</script>

<div class="container">
	<div
		class="drop-zone"
		class:active={isDragging}
		ondragover={handleDragOver}
		ondragleave={handleDragLeave}
		ondrop={handleDrop}>
		{#if isProcessing}
			<p>Processing image... {progress}%</p>
			<div class="progress-bar">
				<div class="progress-fill" style={`width: ${progress}%`} />
			</div>
		{:else}
			<p>Drag and drop an image here, or click to upload</p>
			<input type="file" accept="image/*" onchange={handleFileInput} disabled={isProcessing} />
		{/if}
	</div>

	{#if error}
		<div class="error-message">{error}</div>
	{/if}

	{#if imageUrl && !isProcessing}
		<img src={imageUrl} alt="Uploaded preview" class="preview-image" />
	{/if}

	{#if extractedText}
		<textarea class="text-result" readonly placeholder="Extracted text will appear here...">
			{extractedText}
		</textarea>
		<button onclick={() => navigator.clipboard.writeText(extractedText)}>
			Copy to Clipboard
		</button>
	{/if}
</div>

<style>
	.container {
		max-width: 800px;
		margin: 2rem auto;
		padding: 1rem;
	}

	.drop-zone {
		border: 2px dashed #ccc;
		border-radius: 1rem;
		padding: 2rem;
		text-align: center;
		transition: all 0.3s ease;
		background: white;
		margin-bottom: 2rem;
	}

	.drop-zone.active {
		border-color: #4caf50;
		background-color: #f8fff8;
	}

	.preview-image {
		max-width: 100%;
		height: auto;
		border-radius: 0.5rem;
		margin-top: 1rem;
	}

	.progress-bar {
		width: 100%;
		height: 20px;
		background: #eee;
		border-radius: 10px;
		margin: 1rem 0;
		overflow: hidden;
	}

	.progress-fill {
		height: 100%;
		background: #4caf50;
		transition: width 0.3s ease;
	}

	.text-result {
		width: 100%;
		min-height: 200px;
		padding: 1rem;
		border: 1px solid #ccc;
		border-radius: 0.5rem;
		margin-top: 1rem;
		font-family: monospace;
	}

	.error-message {
		color: #ff4444;
		margin: 1rem 0;
	}
</style>
