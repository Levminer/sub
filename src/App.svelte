<div>
	<div class="bg-gray-700 p-10 rounded-xl m-10 flex justify-evenly">
		<div>
			<h2>Original subtitles</h2>
			<div class="mt-5">
				<button class="button" on:click={originalSub}>Choose file</button>
				<input class="button hidden" type="file" id="originalSub" on:change={() => loadFile(event, 0)} />
			</div>
		</div>
		<div>
			<h2>Machine subtitles</h2>
			<div class="mt-5">
				<button class="button" on:click={machineSub}>Choose file</button>
				<input class="button hidden" type="file" id="machineSub" on:change={() => loadFile(event, 1)} />
			</div>
		</div>
		<div>
			<h2>Translated subtitles</h2>
			<div class="mt-5">
				<button class="button" on:click={downloadTranslated}>Download</button>
			</div>
		</div>
	</div>

	<div class="elements bg-gray-700 m-10 rounded-xl flex flex-row">
		<div class="originalElements w-2/5">
			<!-- original -->
			<h1 class="p-5">Original</h1>
		</div>
		<div class="machineElements w-1/3">
			<!-- machine -->
			<h1 class="p-5">Machine</h1>
		</div>
		<div class="translatedElements w-2/5">
			<!-- translated -->
			<h1 class="p-5">Translated</h1>
		</div>
	</div>
</div>

<script lang="ts">
	import SRTParser from "srt-parser-2"
	import fileSaver from "file-saver"

	const parser = new SRTParser()
	let file: { id: string; startTime: string; endTime: string; text: string }[]

	const originalSub = () => {
		document.querySelector<HTMLButtonElement>("#originalSub").click()
	}

	const machineSub = () => {
		document.querySelector<HTMLButtonElement>("#machineSub").click()
	}

	const downloadTranslated = () => {
		let saveFile = ""

		for (let i = 0; i < file.length; i++) {
			const translatedText = document.querySelector<HTMLInputElement>(`#translated${i}`).value

			if (translatedText !== "") {
				file[i].text = translatedText
			}

			saveFile += `${file[i].id}\n${file[i].startTime} --> ${file[i].endTime}\n${file[i].text}\n\n`
		}

		var blob = new Blob([saveFile], { type: "text/plain;charset=utf-8" })
		fileSaver.saveAs(blob, "translated.srt")
	}

	const loadFile = (event, id) => {
		if (window.FileReader) {
			// Read file
			const reader = new FileReader()
			reader.readAsText(event.target.files[0])

			// File read successful
			reader.onload = (event) => {
				if (id === 0) {
					processData(event.target.result.toString())
				} else {
					machineTranslated(event.target.result.toString())
				}
			}

			// File read error
			reader.onerror = (event) => {
				if (event.target.error.name == "NotReadableError") {
					alert("Failed to upload the file! You uploaded a corrupted or not supported file!")
				}
			}
		} else {
			alert("Your browser doesn't support this function! Tyr another browser!")
		}
	}

	const processData = (data: string) => {
		let parsedData = parser.fromSrt(data)

		file = parsedData

		//console.log(parsedData)

		for (let i = 0; i < parsedData.length; i++) {
			const div = document.createElement("div")

			let cpl = ""

			if (parsedData[i].text.includes("\n")) {
				let lines = parsedData[i].text.split("\n")

				cpl = `${lines[0].length}<br>${lines[1].length}`
			} else {
				cpl = parsedData[i].text.length.toString()
			}

			div.innerHTML = `
			<div class="p-5 flex justify-start items-center gap-3">
				<div class="mr-10">
					<h4>${parsedData[i].id}</h4>
					<h5>${parsedData[i].startTime}</h5>
					<h5>${parsedData[i].endTime}</h5>
				</div>	
				<div class="flex justify-center flex-col items-center mr-5">
					<h5>CPL</h5>
					<h5>${cpl}</h5>
				</div>
				<textarea class="input" cols="2" rows="2">${parsedData[i].text}</textarea>
			</div>
			`

			div.classList.add("originalElement")
			document.querySelector(".originalElements").appendChild(div)
		}

		// Create translated elements
		for (let i = 0; i < parsedData.length; i++) {
			const div = document.createElement("div")

			div.innerHTML = `
			<div class="flex justify-start items-center gap-3 py-5">
				<div class="flex justify-center flex-col items-center mr-5">
					<h5>CPL</h5>
					<h5 id="translatedCPL${i}">0</h5>
				</div>	
				<textarea id="translated${i}" class="input" cols="2" rows="2"></textarea>
			</div>
			`

			div.classList.add("translationElement")
			document.querySelector(".translatedElements").appendChild(div)

			document.querySelector<HTMLTextAreaElement>(`#translated${i}`).addEventListener("keyup", () => {
				let inputText = document.querySelector<HTMLInputElement>(`#translated${i}`).value
				let cpl = ""

				if (inputText === "") {
					inputText = "0"
				}

				if (inputText.includes("\n")) {
					let lines = inputText.split("\n")

					cpl = `${lines[0].length}<br>${lines[1].length}`
				} else {
					cpl = inputText.length.toString()
				}

				document.querySelector(`#translatedCPL${i}`).innerHTML = cpl
			})
		}
	}

	const machineTranslated = (data: string) => {
		let parsedData = parser.fromSrt(data)

		// console.log(parsedData)

		for (let i = 0; i < parsedData.length; i++) {
			const div = document.createElement("div")

			div.innerHTML = `
			<div class="flex justify-start items-center gap-3 py-5">	
				<textarea class="input" cols="2" rows="2">${parsedData[i].text}</textarea>
			</div>
			`

			div.classList.add("machineElement")
			document.querySelector(".machineElements").appendChild(div)
		}
	}
</script>
