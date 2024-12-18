<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Translation Tool with Delete Feature</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: var(--background-color, #f4f4f9);
            color: #333;
            text-align: center;
            padding: 20px;
        }
        h1 {
            color: #444;
        }
        textarea, input {
            width: 80%;
            margin: 10px 0;
            padding: 10px;
            font-size: 16px;
            border: 1px solid #ccc;
            border-radius: 5px;
        }
        button {
            font-size: 16px;
            padding: 10px 20px;
            background-color: #5c85d6;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            margin: 5px;
        }
        button:hover {
            background-color: #4a6cb0;
        }
        table {
            width: 80%;
            margin: 20px auto;
            border-collapse: collapse;
            border: 1px solid #ccc;
        }
        th, td {
            border: 1px solid #ccc;
            padding: 10px;
            text-align: left;
        }
        th {
            background-color: #f4f4f9;
        }
        .output {
            margin-top: 20px;
            font-size: 18px;
            color: #555;
            white-space: pre-wrap;
        }
        .admin-controls {
            display: none;
        }
        .instant-lookup, .search-bar {
            margin-top: 20px;
        }
    </style>
</head>
<body>
    <h1 id="mainTitle">Translation Tool</h1>

    <!-- General Mode Section -->
    <div class="instant-lookup">
        <input id="instantLookupInput" placeholder="Type English text to translate..." />
        <div id="instantLookupOutput"></div>
    </div>

    <div class="search-bar">
        <input id="searchBar" placeholder="Search translations..." />
        <button id="searchButton">Search</button>
    </div>

    <h2>Stored Translations (Dictionary)</h2>
    <table id="translationTable">
        <thead>
            <tr>
                <th>English</th>
                <th>Runic Translation</th>
                <th class="admin-controls">Actions</th>
            </tr>
        </thead>
        <tbody></tbody>
    </table>

    <div>
        <button id="exportDictionaryButton">Export Dictionary</button>
    </div>

    <!-- Admin Mode Section -->
    <div id="adminSection" class="admin-controls">
        <h2>Admin Controls</h2>
        <textarea id="inputText" placeholder="Type English text..."></textarea><br>
        <button id="generateButton">Generate Phonics</button>
        <div id="output"></div>
        <textarea id="manualTranslation" placeholder="Refine translation..."></textarea><br>
        <button id="saveButton">Save Translation</button>
    </div>

    <!-- Mode Toggle -->
    <div id="modeToggle">
        <p>Currently in <span id="currentMode">Viewing Mode</span></p>
        <button id="loginButton">Switch to Admin Mode</button>
        <button id="logoutButton" style="display: none;">Switch to Viewing Mode</button>
    </div>

    <script>
        const translations = {
            "example": "Ah-Ver-Za-Met-El"
        };
        const runicPhonics = {
            "A": "La", "B": "Met", "C": "Ah", "D": "Ver", "E": "Sa", "F": "Ki", "G": "Geh",
            "H": "Deh", "I": "Sa", "J": "Thra", "K": "Ah", "L": "Se", "M": "Hieth", "N": "Et",
            "O": "Th", "P": "Ber", "Q": "Mah", "R": "Im", "S": "To", "T": "Neh", "U": "Keh",
            "V": "Ot", "W": "Ot", "X": "To", "Y": "Ry", "Z": "To"
        };
        const adminPIN = "2359";
        let isAdmin = false;

        // Generate phonics from input
        document.getElementById("generateButton").addEventListener("click", () => {
            const input = document.getElementById("inputText").value.trim();
            if (!input) return alert("Input is empty.");
            const result = [...input.toUpperCase()]
                .map(char => runicPhonics[char] || "")
                .filter((phoneme, i, arr) => phoneme && phoneme !== arr[i - 1])
                .join("-");
            document.getElementById("output").textContent = `Phonics: ${result}`;
            document.getElementById("manualTranslation").value = result;
        });

        // Save translation
        document.getElementById("saveButton").addEventListener("click", () => {
            const english = document.getElementById("inputText").value.trim();
            const runic = document.getElementById("manualTranslation").value.trim();
            if (!english || !runic) return alert("Both English and translation required.");
            translations[english] = runic;
            updateTable();
        });

        // Delete translation
        function deleteTranslation(english) {
            delete translations[english];
            updateTable();
        }

        // Instant lookup
        document.getElementById("instantLookupInput").addEventListener("input", (event) => {
            const input = event.target.value.trim().toLowerCase();
            if (translations[input]) {
                document.getElementById("instantLookupOutput").textContent = `Translation: ${translations[input]}`;
            } else if (input) {
                document.getElementById("instantLookupOutput").textContent = "No saved translation found.";
            } else {
                document.getElementById("instantLookupOutput").textContent = "";
            }
        });

        // Export dictionary
        document.getElementById("exportDictionaryButton").addEventListener("click", () => {
            const dataStr = "data:text/json;charset=utf-8," + encodeURIComponent(JSON.stringify(translations, null, 2));
            const downloadAnchor = document.createElement("a");
            downloadAnchor.setAttribute("href", dataStr);
            downloadAnchor.setAttribute("download", "dictionary.json");
            document.body.appendChild(downloadAnchor);
            downloadAnchor.click();
            downloadAnchor.remove();
        });

        // Update table
        function updateTable() {
            const tbody = document.getElementById("translationTable").querySelector("tbody");
            tbody.innerHTML = "";
            for (const [english, runic] of Object.entries(translations)) {
                const row = document.createElement("tr");
                row.innerHTML = `
                    <td>${english}</td>
                    <td>${runic}</td>
                    ${isAdmin ? `<td><button onclick="deleteTranslation('${english}')">Delete</button></td>` : ""}
                `;
                tbody.appendChild(row);
            }
        }

        // Toggle Admin Mode
        document.getElementById("loginButton").addEventListener("click", () => {
            const pin = prompt("Enter Admin PIN:");
            if (pin === adminPIN) {
                isAdmin = true;
                document.getElementById("currentMode").textContent = "Admin Mode";
                document.getElementById("loginButton").style.display = "none";
                document.getElementById("logoutButton").style.display = "inline-block";
                document.querySelectorAll(".admin-controls").forEach(el => el.style.display = "block");
            } else {
                alert("Incorrect PIN. Access Denied.");
            }
            updateTable();
        });

        document.getElementById("logoutButton").addEventListener("click", () => {
            isAdmin = false;
            document.getElementById("currentMode").textContent = "Viewing Mode";
            document.getElementById("loginButton").style.display = "inline-block";
            document.getElementById("logoutButton").style.display = "none";
            document.querySelectorAll(".admin-controls").forEach(el => el.style.display = "none");
            updateTable();
        });

        updateTable();
    </script>
</body>
</html>
