# Download PDF file from TED.com
Download the PDF file from TED - step by step by yourself

TED.com provides videos and is translated into multiple languages so users can watch them online. It's great for people who want to brush up on the language.<br>
However, we need to listen - speak - read - write. So we need PDF to READ and WRITE.<br>
My girlfriend is learning Chinese and the books they sell to practice reading and writing are quite expensive so I found a way to create PDF files from TED videos.<br>
Let's begin!

# Result
https://www.ted.com/talks/max_tegmark_how_to_keep_ai_under_control/transcript?subtitle=vi
![image](https://github.com/TrumLoi/download_pdf_from_ted.com/assets/66402250/c53ee36c-f0b3-482e-86f3-bd20ff4d2d55)
![image](https://github.com/TrumLoi/download_pdf_from_ted.com/assets/66402250/0649774f-5622-4ff0-b53b-f4802eb5be82)
[如何让 AI 处于掌控之中.pdf](https://github.com/TrumLoi/download_pdf_from_ted.com/files/14627979/AI.pdf)


The way to create a PDF file is by collecting the content in the Subtitle on the video and the Read Transcript section and putting it in one file to export.
- Collect: we can use jQuery from "Custom JavaScript for Websites 2" extension (Run custom JavaScript on any website)
- Export PDF file: we can use the "pdfmake" (on the client side - is on your computer not from Ted website)
- Subtitle: is your language
- Read Transcript: is the language you learn

## 1) Install the "Custom JavaScript for Websites 2" extension on Chrome:
We need to get content on the Subtitle and the Read Transcript section so we need javascript works on ted.com <br>
I found that "Custom JavaScript for Websites 2" can do that. Just install it and set up to use.<br>
Go to the Chrome store and install it: https://chromewebstore.google.com/detail/custom-javascript-for-web/ddbjnfjiigjmcpcpkmhogomapikjbjdk <br>
After installation, we have an extension here:<br>
![image](https://github.com/TrumLoi/download_pdf_from_ted.com/assets/66402250/ea2209d9-30d6-4705-9f16-3efbf1e3c3fc)
![image](https://github.com/TrumLoi/download_pdf_from_ted.com/assets/66402250/d3581ceb-cf89-48ab-a9cc-768fa3a116c5)

## 2) Choose to use jQuery's latest for "Custom JavaScript for Websites 2":
![image](https://github.com/TrumLoi/download_pdf_from_ted.com/assets/66402250/d6405d43-8778-4a7b-aed9-c1b1603bafcc)

## 3) Import the "pdfmake" library:
Import the "pdfmake" library from [cdnjs.com](https://cdnjs.com/libraries/pdfmake): `https://cdnjs.cloudflare.com/ajax/libs/pdfmake/0.2.10/pdfmake.js`
![image](https://github.com/TrumLoi/download_pdf_from_ted.com/assets/66402250/601cf5e4-12c5-403a-9a37-2aae042bb4e9)

## 4) Use the jQuery code to get content and export to PDF file:
Copy and paste to the script content (I will explain the code at the end)
```
let result = [];
let crVideoTitle;
let crMainLanguage;
let crTransLanguage;
let videoTime;
let timeSession = [];
timeSession['main'] = [];
timeSession['trans'] = [];

$("body").on('DOMSubtreeModified', "#ted-player", function() {
	videoTitle = $('#talk-title').text();
	if (crVideoTitle === undefined && videoTitle !== undefined) {
		crVideoTitle = videoTitle;
	}
	let subtitle = $('.transition-transform .text-textPrimary-onDark').text();
	let trans = $('[data-testid=paragraphs-container] div.bg-yellow-500').text();
	subtitle = subtitle.replace(/\n|\r/g, " ");
	trans = trans.replace(/\n|\r/g, " ");
	let time;
	if (trans) {
		time = $('[data-testid=paragraphs-container] div.bg-yellow-500').parent().parent().prev().text();
	}

	if (time !== undefined && videoTime === undefined) {
		videoTime = time;
	}

	if (time !== videoTime) {
		videoTime = time;
		if (timeSession && timeSession['main'].length > 0) {
			// (Applause)
			if (!(timeSession['main'].length === 1 &&
					timeSession['main'][0].startsWith('(') &&
					timeSession['main'][0].endsWith(')'))) {
				result.push(timeSession);
				console.log("result: ", result);
			}
			timeSession = [];
			timeSession['main'] = [];
			timeSession['trans'] = [];
		}
	}

	if (subtitle !== crMainLanguage && trans !== crTransLanguage && subtitle.length > 0 && trans.length > 0) {
		crMainLanguage = subtitle;
		crTransLanguage = trans;
		timeSession['main'].push(subtitle);
		timeSession['trans'].push(trans);
	}

	if (crVideoTitle !== undefined && crVideoTitle !== videoTitle) {
		if (result.length > 0 && confirm("Có xuất file không?") == true) {
			pdfMakeFileAndDownload(result, crVideoTitle);
		}
		crVideoTitle = undefined;
		result = [];
	}
});

function pdfMakeFileAndDownload(exportArr, fileName) {
	let docDefinition = pdfDocDefinition(exportArr, fileName);
	pdfMake.fonts = {
		Roboto: {
			normal: 'https://cdnjs.cloudflare.com/ajax/libs/pdfmake/0.1.66/fonts/Roboto/Roboto-Regular.ttf',
			bold: 'https://cdnjs.cloudflare.com/ajax/libs/pdfmake/0.1.66/fonts/Roboto/Roboto-Medium.ttf',
			italics: 'https://cdnjs.cloudflare.com/ajax/libs/pdfmake/0.1.66/fonts/Roboto/Roboto-Italic.ttf',
			bolditalics: 'https://cdnjs.cloudflare.com/ajax/libs/pdfmake/0.1.66/fonts/Roboto/Roboto-MediumItalic.ttf'
		},
		cn: {
			normal: 'https://cdnjs.cloudflare.com/ajax/libs/chiron-sans-hk/2.046/ttf/ChironSansHKTT-Normal.ttf',
			bold: 'https://cdnjs.cloudflare.com/ajax/libs/chiron-sans-hk/2.046/ttf/ChironSansHKTT-Medium.ttf',
			italics: 'https://cdnjs.cloudflare.com/ajax/libs/chiron-sans-hk/2.046/ttf/ChironSansHKTT-Regular.ttf',
			bolditalics: 'https://cdnjs.cloudflare.com/ajax/libs/chiron-sans-hk/2.046/ttf/ChironSansHKTT-Medium.ttf'
		}
	};
	pdfMake.createPdf(docDefinition).download(fileName + '.pdf');
}

function makePDFContent(exportArr) {
	let numRows = exportArr.length;
	return exportArr.map(function(arr, index) {
		let column2 = index === 0 ? {
			text: '',
			rowSpan: numRows
		} : '';
		return [{
				text: [{
						text: arr['trans'].join(' ') + '\n',
						style: 'mainLanguage'
					},
					{
						text: arr['main'].join(' '),
						style: 'transLanguage'
					}
				],
				margin: [0, 0, 0, 20]
			},
			column2
		]
	});
}

function pdfDocDefinition(exportArr, title) {
	const bodyContent = makePDFContent(exportArr);
	console.log("bodyContent: ", bodyContent);
	return {
		content: [{
				text: title,
				style: 'header'
			},
			{
				style: 'tableExample',
				table: {
					widths: ['*', '30%'],
					body: bodyContent
				},
				layout: 'noBorders'
			}
		],
		styles: {
			header: {
				fontSize: 18,
				bold: true,
				margin: [0, 0, 0, 10]
			},
			tableExample: {
				margin: [0, 0, 0, 0]
			},
			mainLanguage: {
				bold: true,
				fontSize: 16,
				color: 'black',
				font: 'cn'
			},
			transLanguage: {
				fontSize: 12,
				italics: true,
				color: 'gray'
			}
		},
		defaultStyle: {
			font: 'cn'
		}
	};
}
```
![image](https://github.com/TrumLoi/download_pdf_from_ted.com/assets/66402250/0914736e-18a8-4117-936f-b712c8bb3404)
After save, reload that Ted website

## 5) Begin make PDF file
- Open the video you want to make the file. Stop at begin (before the transcript runs - if the transcript runs you can see the highlight yellow on the text at the Read transcript session)<br>
- Click "Read transcript" and choose the language want to learn.
![image](https://github.com/TrumLoi/download_pdf_from_ted.com/assets/66402250/467235c1-1f07-4004-bc8a-32278454a7f7)

- Click and choose Subtitle on the video and make sure the "On" switch is on<br>
![image](https://github.com/TrumLoi/download_pdf_from_ted.com/assets/66402250/a4e70a0d-adb1-41f7-9a49-cfa31fa557bf)
- Switch Autoplay to on (you may need setting speed to 1.5 to progress faster)<br>
![image](https://github.com/TrumLoi/download_pdf_from_ted.com/assets/66402250/ee2090f9-fd70-45b2-a044-a82dc11b5109)
- Play the video and wait for video to end. At the end, this code will show a confirm message. Confirm to export file.<br>
(You can view progress collect content in Console log - press F12 and select "Console" tab )<br>
![image](https://github.com/TrumLoi/download_pdf_from_ted.com/assets/66402250/5e46e284-e5a9-4c07-ad95-db61cea150dc)
Save<br>
![image](https://github.com/TrumLoi/download_pdf_from_ted.com/assets/66402250/9ecb6b31-43af-4e98-a9bd-69c0e87fdb12)
Result<br>
![image](https://github.com/TrumLoi/download_pdf_from_ted.com/assets/66402250/2b162cd9-bc18-4112-a4ee-faf4c7c05252)

# EXPLAIN CODE:
See you next time.








