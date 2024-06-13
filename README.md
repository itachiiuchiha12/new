<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>File Management System (FMS)</title>

    <style>
        body {
            background-color: #f8f9fa;
        }

        .card {
            border-radius: 10px;
            box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
        }

        .card-header {
            background-color: #007bff;
            color: #fff;
            border-radius: 10px 10px 0 0;
        }

        .card-body {
            padding: 2rem;
        }

        .form-group {
            margin-bottom: 1.5rem;
        }

        .btn-primary {
            background-color: #007bff;
            border: none;
        }

        .btn-primary:hover {
            background-color: #0056b3;
        }

        .btn-secondary,
        .btn-danger {
            margin-left: 10px;
        }

        .btn-block {
            width: 100%;
        }

        .jsonData {
            margin-top: 20px;
        }
    </style>
    <link href="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/css/bootstrap.min.css" rel="stylesheet">
</head>

<body>

    <div class="container mt-5">
        <div class="row justify-content-center">
            <div class="col-md-8">
                <!-- Upload CSV Block -->
                <div class="card mb-4">
                    <div class="card-header text-center">
                        <h3>Upload CSV File</h3>
                    </div>
                    <div class="card-body">
                        <div class="form-group">
                            <label for="csvFileInput">Choose CSV File</label>
                            <input type="file" class="form-control-file" id="csvFileInput" accept=".csv">
                        </div>
                        <button id="uploadBtn" class="btn btn-primary btn-block">Upload and Convert</button>
                        <button type="button" id="nextEntry" class="btn btn-primary mt-4 d-none">Next Entry</button>
                        <div id="jsonData" class="jsonData mt-3"></div>
                    </div>
                </div>

                <!-- File Management Form -->
                <div class="card">
                    <div class="card-header text-center">
                        <h2>File Management System (FMS)</h2>
                    </div>
                    <div class="card-body">
                        <form id="fileForm">
                            <div class="form-group">
                                <label for="filename">File Name</label>
                                <input type="text" class="form-control" id="filename" required>
                            </div>
                            <div class="form-group">
                                <label for="fromDate">From Date</label>
                                <input type="text" class="form-control" id="fromDate">
                            </div>
                            <div class="form-group">
                                <label for="toDate">To Date</label>
                                <input type="text" class="form-control" id="toDate">
                            </div>
                            <div class="form-group">
                                <label for="shelf">Shelf</label>
                                <input type="text" class="form-control" id="shelf" required>
                            </div>
                            <div class="form-group">
                                <label for="fileNo">File No</label>
                                <input type="text" class="form-control" id="fileNo" required>
                            </div>
                            <div class="form-group">
                                <label for="year">Year</label>
                                <input type="text" class="form-control" id="year">
                            </div>
                            <div class="form-group">
                                <label for="location">Location</label>
                                <input type="text" class="form-control" id="location">
                            </div>
                            <div class="text-center">
                                <button type="button" id="submitData" class="btn btn-primary">Submit</button>
                                <button type="button" id="editData" class="btn btn-secondary">Update</button>
                                <button type="button" id="retrieveData" class="btn btn-primary">Retrieve</button>
                                <button type="button" id="deleteData" class="btn btn-danger">Delete</button>
                            </div>
                        </form>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/10.12.2/firebase-app.js";
        import { getDatabase, ref, set, child, get, update, remove } from "https://www.gstatic.com/firebasejs/10.12.2/firebase-database.js";

        // Your web app's Firebase configuration
        const firebaseConfig = {
            apiKey: "AIzaSyAOlfx4IZbnKTE-pOElBvkySj_iNtZ1l4w",
            authDomain: "fir-v1-aa90c.firebaseapp.com",
            databaseURL: "https://fir-v1-aa90c-default-rtdb.firebaseio.com",
            projectId: "fir-v1-aa90c",
            storageBucket: "fir-v1-aa90c.appspot.com",
            messagingSenderId: "1081395760776",
            appId: "1:1081395760776:web:cec5112d07762badead93d"
        };

        // Initialize Firebase
        const app = initializeApp(firebaseConfig);
        const db = getDatabase();

        let jsonData = []; // To store CSV data in JSON format
        let currentIndex = 0; // To keep track of current index in JSON data

        document.getElementById('uploadBtn').addEventListener('click', handleCsvUpload);
        document.getElementById('nextEntry').addEventListener('click', fillFormWithNextEntry);
        document.getElementById('submitData').addEventListener('click', addFileData);
        document.getElementById('retrieveData').addEventListener('click', retrieveData);
        document.getElementById('editData').addEventListener('click', updateFileData);
        document.getElementById('deleteData').addEventListener('click', deleteFileData);

        function handleCsvUpload() {
            const fileInput = document.getElementById('csvFileInput');
            const file = fileInput.files[0];

            if (file) {
                const reader = new FileReader();
                reader.onload = function (e) {
                    const csv = e.target.result;
                    jsonData = csvToJson(csv);
                    displayNextEntryButton(); // Show next entry button
                    fillFormWithNextEntry(); // Fill form with first entry
                };
                reader.readAsText(file);
            } else {
                alert('Please select a CSV file to upload.');
            }
        }

        function csvToJson(csv) {
            const lines = csv.split('\n');
            const result = [];
            const headers = lines[0].split('\t').map(header => header.trim()); // Assuming tab-separated values

            for (let i = 1; i < lines.length; i++) {
                const obj = {};
                const currentline = lines[i].split('\t');

                for (let j = 0; j < headers.length; j++) {
                    obj[headers[j]] = currentline[j] || ''; // Assign value or empty string if undefined
                }
                result.push(obj);
            }
            return result;
        }



        function displayNextEntryButton() {
            document.getElementById('nextEntry').classList.remove('d-none');
        }

        function fillFormWithNextEntry() {
            if (currentIndex < jsonData.length) {
                var entry = jsonData[currentIndex];
                console.table(entry);
                document.getElementById('filename').value = entry.fileName;
                document.getElementById('fromDate').value = entry.from;
                document.getElementById('toDate').value = entry.to;
                document.getElementById('shelf').value = entry.shelfNo;
                document.getElementById('fileNo').value = entry.fileN;
                document.getElementById('year').value = entry.year;
                document.getElementById('location').value = entry.location;
                currentIndex++;
            } else {
                alert('All entries reviewed.');
            }
        }

        // Your other Firebase functions (add, update, retrieve, delete) remain unchanged
        function addFileData() {
            const fileName = document.getElementById('filename').value;
            const fromDate = document.getElementById('fromDate').value;
            const toDate = document.getElementById('toDate').value;
            const shelf = document.getElementById('shelf').value;
            const fileNo = document.getElementById('fileNo').value;
            const year = document.getElementById('year').value;
            const location = document.getElementById('location').value;

            set(ref(db, 'FileSet/' + fileName), {
                FileName: fileName,
                FromDate: fromDate,
                ToDate: toDate,
                Shelf: shelf,
                FileNo: fileNo,
                Year: year,
                Location: location
            }).then(() => {
                alert('Data Successfully Added!');
            }).catch((error) => {
                alert('Error: ' + error.message);
            });
        }

        function retrieveData() {
            const fileName = document.getElementById('filename').value.toUpperCase();
            const dbRef = ref(db);

            get(child(dbRef, 'FileSet/' + fileName)).then((snapshot) => {
                if (snapshot.exists()) {
                    document.getElementById('fromDate').value = snapshot.val().FromDate;
                    document.getElementById('toDate').value = snapshot.val().ToDate;
                    document.getElementById('shelf').value = snapshot.val().Shelf;
                    document.getElementById('fileNo').value = snapshot.val().FileNo;
                    document.getElementById('year').value = snapshot.val().Year;
                    document.getElementById('location').value = snapshot.val().Location;
                } else {
                    alert('File does not exist!');
                }
            }).catch((error) => {
                alert('Error: ' + error.message);
            });
        }

        function updateFileData() {
            const fileName = document.getElementById('filename').value.toUpperCase();
            const fromDate = document.getElementById('fromDate').value.toUpperCase();
            const toDate = document.getElementById('toDate').value.toUpperCase();
            const shelf = document.getElementById('shelf').value.toUpperCase();
            const fileNo = document.getElementById('fileNo').value.toUpperCase();
            const year = document.getElementById('year').value.toUpperCase();
            const location = document.getElementById('location').value.toUpperCase();

            update(ref(db, 'FileSet/' + fileName), {
                FileName: fileName,
                FromDate: fromDate,
                ToDate: toDate,
                Shelf: shelf,
                FileNo: fileNo,
                Year: year,
                Location: location
            }).then(() => {
                alert('Data Successfully Updated!');
            }).catch((error) => {
                alert('Error: ' + error.message);
            });
        }

        function deleteFileData() {
            const fileName = document.getElementById('filename').value.toUpperCase();
            remove(ref(db, 'FileSet/' + fileName)).then(() => {
                alert('Data Successfully Deleted!');
            }).catch((error) => {
                alert('Error: ' + error.message);
            });
        }

    </script>
    <script src="https://code.jquery.com/jquery-3.5.1.min.js"></script>
    <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/js/bootstrap.min.js"></script>
</body>

</html>
