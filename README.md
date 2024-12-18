<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Mailing List Tool</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.17.0/xlsx.full.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.4.0/jspdf.umd.min.js"></script>
    <style>
        body {
            font-family: Arial, sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            margin: 0;
            padding: 20px;
            background-color: #f5f7fa;
            color: #333;
        }
        h1, h2 {
            text-align: center;
        }
        .button {
            padding: 8px 16px;
            margin: 10px;
            font-size: 1em;
            background-color: #1a1a2e;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
        }
        textarea {
            width: 100%;
            height: 100px;
            margin-top: 10px;
        }
        .container {
            max-width: 600px;
            width: 100%;
            margin: 20px;
            padding: 20px;
            border: 1px solid #ccc;
            border-radius: 5px;
            background-color: white;
        }
        .label-option {
            display: flex;
            align-items: center;
            margin-top: 10px;
        }
        .label-option input {
            margin-right: 8px;
        }
        #contact-list {
            margin-top: 10px;
        }
        .contact-info {
            padding: 5px 0;
            border-bottom: 1px solid #ddd;
        }
    </style>
</head>
<body>
    <h1>Mailing List Tool</h1>
    <input type="file" id="file-input" accept=".xlsx, .xls">
    <button class="button" onclick="handleFile()">Upload File</button>

    <!-- Contact List Section -->
    <div class="container">
        <h2>Contact List</h2>
        <button class="button" onclick="generateContactList()">Generate Contact List</button>
        <button class="button" onclick="generatePDF('contact-list', 'Contact_List')">Print Contact List PDF</button>
        <div id="contact-list">
            <p>Upload a file and click "Generate Contact List" to view contacts here.</p>
        </div>
    </div>

    <!-- Letter Template Section -->
    <div class="container">
        <h2>Letter Template</h2>
        <textarea id="template">
Dear {Contact},

We are interested in discussing your property at {Address}, {City}, {State}, {Zipcode}.

Best regards,
[Your Company Name]
        </textarea>
        <button class="button" onclick="generateLetters()">Generate Letters</button>
        <button class="button" onclick="generatePDF('letters-content', 'Letters', true)">Print Letters PDF</button>
    </div>

    <!-- Label Maker Section -->
    <div class="container">
        <h2>Label Maker</h2>
        <div class="label-option">
            <input type="checkbox" id="label-checkbox">
            <label for="label-checkbox">Generate Labels Instead of Letters</label>
        </div>
        <button class="button" onclick="generateLabelsPDF()">Print Labels PDF</button>
    </div>

    <!-- Letters Content -->
    <div id="letters-content" style="display:none;"></div>

    <script>
        let contacts = [];

        function handleFile() {
            const fileInput = document.getElementById("file-input").files[0];
            if (!fileInput) {
                alert("Please upload a file.");
                return;
            }
            const reader = new FileReader();
            reader.onload = function(event) {
                try {
                    const data = new Uint8Array(event.target.result);
                    const workbook = XLSX.read(data, { type: "array" });
                    const sheet = workbook.Sheets[workbook.SheetNames[0]];
                    contacts = XLSX.utils.sheet_to_json(sheet, { header: 1 });
                    alert("Contacts loaded successfully!");
                } catch (error) {
                    console.error("Error reading file:", error);
                }
            };
            reader.readAsArrayBuffer(fileInput);
        }

        function generateContactList() {
            if (contacts.length <= 1) {
                alert("Please upload a contact list first.");
                return;
            }
            const contactList = document.getElementById("contact-list");
            contactList.innerHTML = "";
            contacts.slice(1).forEach(row => {
                const contact = row[0] || "";
                const address = row[1] || "";
                if (!contact || !address) return; // Skip rows with blank Contact or Address

                const contactInfo = document.createElement("div");
                contactInfo.classList.add("contact-info");
                contactInfo.innerHTML = `
                    <strong>Contact:</strong> ${contact}<br>
                    <strong>Address:</strong> ${address}<br>
                    <strong>City:</strong> ${row[2] || "No City"}<br>
                    <strong>State:</strong> ${row[3] || "No State"}<br>
                    <strong>Zipcode:</strong> ${row[4] || "No Zip"}<br>
                    <strong>Phone:</strong> ${row[5] || "No Phone"}
                `;
                contactList.appendChild(contactInfo);
            });
        }

        function generateLetters() {
            if (contacts.length <= 1) {
                alert("Please upload a contact list first.");
                return;
            }
            const template = document.getElementById("template").value;
            const lettersContent = document.getElementById("letters-content");
            lettersContent.innerHTML = "";
            contacts.slice(1).forEach(row => {
                const contact = row[0] || "";
                const address = row[1] || "";
                if (!contact || !address) return; // Skip rows with blank Contact or Address

                const city = row[2] || "No City";
                const state = row[3] || "No State";
                const zipcode = row[4] || "No Zip";
                const letter = template
                    .replace("{Contact}", contact)
                    .replace("{Address}", address)
                    .replace("{City}", city)
                    .replace("{State}", state)
                    .replace("{Zipcode}", zipcode);
                const letterElement = document.createElement("pre");
                letterElement.textContent = letter;
                lettersContent.appendChild(letterElement);
            });
            lettersContent.style.display = "block";
        }

        async function generateLabelsPDF() {
            const { jsPDF } = window.jspdf;
            const doc = new jsPDF();
            const margin = 10;
            const labelWidth = 65;
            const labelHeight = 25;
            let x = margin, y = margin;

            contacts.slice(1).forEach((row, index) => {
                const contact = row[0] || "";
                const address = row[1] || "";
                if (!contact || !address) return; // Skip rows with blank Contact or Address

                const city = row[2] || "No City";
                const state = row[3] || "No State";
                const zipcode = row[4] || "No Zip";
                const labelText = `${contact}\n${address}\n${city}, ${state} ${zipcode}`;
                const lines = doc.splitTextToSize(labelText, labelWidth - 2);

                lines.forEach((line, lineIndex) => {
                    doc.text(line, x, y + lineIndex * 6);
                });

                y += labelHeight;

                if (y + labelHeight > 290) {
                    y = margin;
                    x += labelWidth;
                    if (x + labelWidth > 210) {
                        doc.addPage();
                        x = margin;
                    }
                }
            });

            doc.save("Labels.pdf");
        }

        async function generatePDF(elementId, fileName, isLetters = false) {
            const { jsPDF } = window.jspdf;
            const doc = new jsPDF();
            const element = document.getElementById(elementId);

            if (isLetters) {
                const letters = element.querySelectorAll("pre");
                letters.forEach((letter, index) => {
                    doc.text(letter.innerText, 10, 10, { maxWidth: 180 });
                    if (index < letters.length - 1) doc.addPage();
                });
            } else {
                const contactText = Array.from(element.querySelectorAll(".contact-info"))
                    .filter(info => info.innerText.trim() !== "") // Skip blank entries
                    .map(info => info.innerText)
                    .join("\n\n");
                doc.text(contactText, 10, 10, { maxWidth: 180 });
            }

            doc.save(`${fileName}.pdf`);
        }
    </script>
</body>
</html>
