<script lang="ts">
	import { CodeBlock, FileDropzone } from '@skeletonlabs/skeleton';
	import { BinaryBitmap, HybridBinarizer, QRCodeReader, RGBLuminanceSource } from '@zxing/library';
	import { Image } from 'image-js';

	const qrReader = new QRCodeReader();
	let files: FileList | undefined = undefined;
	let imageSrc: string = '';
	let decodedText: string = '';
	let errorMessage: string = '';

	function resetStates() {
		imageSrc = '';
		decodedText = '';
		errorMessage = '';
		files = undefined;
	}

	async function handleFileChange(event: Event) {
		resetStates();
		const input = event.target as HTMLInputElement;
		if (input && input.files && input.files.length > 0) {
			const file = input.files[0];
			if (file && file instanceof File) {
				imageSrc = URL.createObjectURL(file);
				decodeQR(file);
			} else {
				console.log('Selected file is not valid.');
			}
		} else {
			console.log('No file selected or input is not an HTMLInputElement.');
		}
	}

	async function decodeQR(file: File) {
		try {
			const arrayBuffer = await file.arrayBuffer();
			const image = await Image.load(new Uint8Array(arrayBuffer));
			console.log('Image loaded successfully');

			// Check the image size and format
			console.log(`Image dimensions: ${image.width}x${image.height}`);

			// Convert to grayscale
			const grayscaleImage = image.grey();
			const pixelData = new Uint8ClampedArray(grayscaleImage.data.buffer);

			// Check grayscale image dimensions
			console.log(`Grayscale dimensions: ${grayscaleImage.width}x${grayscaleImage.height}`);

			// Create luminance source and binary bitmap
			const source = new RGBLuminanceSource(pixelData, grayscaleImage.width, grayscaleImage.height);
			const binaryBitmap = new BinaryBitmap(new HybridBinarizer(source));

			// Decode the QR code
			const result = qrReader.decode(binaryBitmap);
			console.log('QRCode Result: ', result.getText());
			decodedText = result.getText();
		} catch (NotFoundException) {
			console.log('Error decoding QR code:', NotFoundException);
			errorMessage = 'This is not a QR Code.';
		}
	}
</script>

<div class="container h-full mx-auto flex flex-col justify-evenly items-center">
	{#if !imageSrc}
		<div class="top-0"><h1 class="text-4xl font-bold mt-1">QR Code Reader</h1></div>
	{:else if errorMessage}
		<h1 class="text-4xl font-bold mt-4">This is not a QR Code.</h1>
	{/if}
	<div class="space-y-10 text-center items-center">
		{#if !imageSrc}
			<FileDropzone name="files" bind:files on:change={handleFileChange} />
		{:else}
			{#if !errorMessage}
				<CodeBlock language="text" code={decodedText}></CodeBlock>
			{/if}
			<div class="relative p-4">
				<button
					on:click={resetStates}
					type="button"
					class="btn-icon btn-icon-sm variant-filled absolute top-0 right-0"
				>
					<svg
						xmlns="http://www.w3.org/2000/svg"
						width="24"
						height="24"
						viewBox="0 0 24 24"
						fill="none"
						stroke="currentColor"
						stroke-width="2"
						stroke-linecap="round"
						stroke-linejoin="round"
						class="lucide lucide-x"><path d="M18 6 6 18" /><path d="m6 6 12 12" /></svg
					>
				</button>

				<img src={imageSrc} alt="" />
			</div>
		{/if}
	</div>
</div>
