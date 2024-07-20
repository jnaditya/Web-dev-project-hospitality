Aditya's Approach to Digitalizing the Hospitality Process Web Application
1. Project Setup
Technology Stack:
Frontend: HTML, CSS, JavaScript, React.js
Backend: Node.js, Express.js
Database: MongoDB (for scalability and ease of use with Node.js)
Others: CSV parsing libraries (e.g., csv-parser for Node.js)
2. User Interface
Home Page:
Upload section for two CSV files: Group Information and Hostel Information.
Submit button to process the files and allocate rooms.
Allocation Results:
Table displaying the allocated rooms.
Button to download the allocation results as a CSV file.
3. CSV Parsing and Data Storage
Use csv-parser or similar library to read and parse the CSV files.
Store parsed data in MongoDB collections for groups and hostels.
4. Room Allocation Algorithm
Input: Parsed data from the two CSV files.

Constraints:

Group members with the same ID should stay together.

Gender-specific accommodations.

Room capacity should not be exceeded.

Steps:

Parse CSV files and store data.
Sort groups by size (larger groups first) to optimize space usage.
Iterate through sorted groups and find suitable rooms based on capacity and gender.
Allocate group members to rooms, ensuring the above constraints are met.
Store allocation results.
5. Backend Implementation
API Endpoints:
/upload: Endpoint to handle file uploads and initiate room allocation.
/allocation: Endpoint to fetch the room allocation results.
/download: Endpoint to download the room allocation results as a CSV file.
6. Frontend Implementation
File Upload Component:

Use React components to handle file input and submission.

Display loading indicator while processing.

Results Display Component:

Use tables to show allocation results.

Provide a download button for the CSV file.

7. Code Snippets
Backend: Express Server
const express = require('express');
const multer = require('multer');
const csv = require('csv-parser');
const fs = require('fs');
const mongoose = require('mongoose');
const app = express();
const upload = multer({ dest: 'uploads/' });

mongoose.connect('mongodb://localhost:27017/hospitality', {
  useNewUrlParser: true,
  useUnifiedTopology: true
});

const GroupSchema = new mongoose.Schema({
  groupId: Number,
  members: Number,
  gender: String
});

const HostelSchema = new mongoose.Schema({
  hostelName: String,
  roomNumber: Number,
  capacity: Number,
  gender: String
});

const AllocationSchema = new mongoose.Schema({
  groupId: Number,
  hostelName: String,
  roomNumber: Number,
  membersAllocated: Number
});

const Group = mongoose.model('Group', GroupSchema);
const Hostel = mongoose.model('Hostel', HostelSchema);
const Allocation = mongoose.model('Allocation', AllocationSchema);

app.post('/upload', upload.array('files', 2), (req, res) => {
  const groupFile = req.files[0].path;
  const hostelFile = req.files[1].path;

  // Parse CSV files and store data in MongoDB
  parseCSV(groupFile, 'Group');
  parseCSV(hostelFile, 'Hostel');

  allocateRooms().then(() => {
    res.send('Files uploaded and processed.');
  });
});

const parseCSV = (filePath, type) => {
  fs.createReadStream(filePath)
    .pipe(csv())
    .on('data', (row) => {
      if (type === 'Group') {
        const group = new Group({
          groupId: row['Group ID'],
          members: row['Members'],
          gender: row['Gender']
        });
        group.save();
      } else if (type === 'Hostel') {
        const hostel = new Hostel({
          hostelName: row['Hostel Name'],
          roomNumber: row['Room Number'],
          capacity: row['Capacity'],
          gender: row['Gender']
        });
        hostel.save();
      }
    });
};

const allocateRooms = async () => {
  const groups = await Group.find().sort({ members: -1 }).exec();
  const hostels = await Hostel.find().exec();

  for (const group of groups) {
    for (const hostel of hostels) {
      if (hostel.gender === group.gender && hostel.capacity >= group.members) {
        const allocation = new Allocation({
          groupId: group.groupId,
          hostelName: hostel.hostelName,
          roomNumber: hostel.roomNumber,
          membersAllocated: group.members
        });
        allocation.save();
        hostel.capacity -= group.members;
        break;
      }
    }
  }
};

app.get('/allocation', async (req, res) => {
  const allocations = await Allocation.find().exec();
  res.json(allocations);
});

app.get('/download', async (req, res) => {
  const allocations = await Allocation.find().exec();
  const csvWriter = require('csv-writer').createObjectCsvWriter({
    path: 'allocations.csv',
    header: [
      { id: 'groupId', title: 'Group ID' },
      { id: 'hostelName', title: 'Hostel Name' },
      { id: 'roomNumber', title: 'Room Number' },
      { id: 'membersAllocated', title: 'Members Allocated' }
    ]
  });
  csvWriter.writeRecords(allocations).then(() => {
    res.download('allocations.csv');
  });
});

app.listen(3000, () => {
  console.log('Server is running on port 3000');
});
Frontend: React Components
import React, { useState } from 'react';
import axios from 'axios';

const FileUpload = () => {
  const [files, setFiles] = useState(null);
  const [allocations, setAllocations] = useState([]);

  const handleFileChange = (e) => {
    setFiles(e.target.files);
  };

  const handleUpload = () => {
    const formData = new FormData();
    for (const file of files) {
      formData.append('files', file);
    }

    axios.post('/upload', formData)
      .then(() => {
        fetchAllocations();
      })
      .catch(err => console.error(err));
  };

  const fetchAllocations = () => {
    axios.get('/allocation')
      .then(response => {
        setAllocations(response.data);
      })
      .catch(err => console.error(err));
  };

  const handleDownload = () => {
    axios.get('/download')
      .then(() => {
        window.location.href = '/download';
      })
      .catch(err => console.error(err));
  };

  return (
    <div>
      <input type="file" multiple onChange={handleFileChange} />
      <button onClick={handleUpload}>Upload</button>

      {allocations.length > 0 && (
        <div>
          <table>
            <thead>
              <tr>
                <th>Group ID</th>
                <th>Hostel Name</th>
                <th>Room Number</th>
                <th>Members Allocated</th>
              </tr>
            </thead>
            <tbody>
              {allocations.map((allocation, index) => (
                <tr key={index}>
                  <td>{allocation.groupId}</td>
                  <td>{allocation.hostelName}</td>
                  <td>{allocation.roomNumber}</td>
                  <td>{allocation.membersAllocated}</td>
                </tr>
              ))}
            </tbody>
          </table>
          <button onClick={handleDownload}>Download CSV</button>
        </div>
      )}
    </div>
  );
};

export default FileUpload;
8. Documentation and Submission
Documentation:

Overview of the project and its purpose.

Detailed steps to run the application (setting up the environment, installing dependencies, running the server).

Explanation of the room allocation algorithm.

Submission:

Create a public repository on GitHub.

Include the source code, documentation, and any additional instructions.

Submit the GitHub link via the provided Google form.

This approach ensures a well-organized, scalable, and user-friendly web application for digitalizing the hospitality process.
