# MongoDB Day-2 Task - Zen Class Programme Database

The database is designed to store information about users, codekata submissions, attendance records, topics, tasks, company_drives, and mentors.

## Table of Contents

1. [Database Collections](#database-collections)
2. [Getting Started](#getting-started)
3. [Queries](#queries)
4. [Contributing](#contributing)
5. [License](#license)

## Database Collections

The database consists of the following collections:

1. `users`: Stores information about program users.
2. `codekata`: Contains codekata submissions and user performance.
3. `attendance`: Records attendance and task submissions by users.
4. `topics`: Stores information about topics taught in the program.
5. `tasks`: Contains details about tasks associated with topics.
6. `company_drives`: Records information about company placement drives.
7. `mentors`: Stores data about program mentors and their mentee counts.

## Getting Started

To set up and use the database, you will need:

- MongoDB installed and running.
- A MongoDB client or shell to execute queries.
- Properly configured connection settings.

## Queries:

### 1. Find all the topics and tasks taught in the month of October:

     ```bash
     db.getCollection('topics').aggregate(
    [
     {
        $lookup: {
          from: 'tasks',
          localField: 'topic',
          foreignField: 'task',
          as: 'Topic Task Data'
        }
      },
      {
        $match: { topicDate: { $regex: '2020-10' } }
      },
     {
        $project: {
          _id: 0,
          topic: 1,
          topicDate: 1,
          'Topic Task Data.userId': 1,
          'Topic Task Data.submitted': 1,
          'Topic Task Data.task': 1
        }
      }
    ],
    {maxTimeMS: 60000, allowDiskUse: true }
    ); 

### 2. Find all the company drives which appeared between 15 oct-2020 and 31-oct-2020:

    ```bash
    db.getCollection('company_drives').find({
     driveDate: {
       $gte: '2020-10-15',
       $lte: '2020-10-31'
     }
    });

### 3. Find all the company drives and students who are appeared for the placement:


    ```bash
    db.getCollection('Company_Drives').aggregate(
    [
       {
      $lookup: {
        from: 'Users',
        localField: 'userId',
        foreignField: 'userId',
        as: 'studentInfo'
      }
    },
    {
      $project: {
        _id: 0,
        'studentInfo.userName': 1,
        company: 1,
        driveDate: 1,
        'studentInfo.userEmail': 1,
        'studentInfo.userId': 1
      }
    }
    ],
    { maxTimeMS: 60000, allowDiskUse: true }
    );

### 4. Find the number of problems solved by the user in codekata:

    ```bash
    db.getCollection('codekata').aggregate(
    [
    {
      $group: {
        _id: 'Total Problem Solved By Users',
        count: { $sum: '$problemSolved' }
      }
    }
    ],
    { maxTimeMS: 60000, allowDiskUse: true }
    );

### 5. Find all the mentors with who has the mentee's count more than 15:

     ```bash
     db.getCollection('mentors').aggregate(
     [{ $match: { menteeCount: { $gt: 15 } } }],
     { maxTimeMS: 60000, allowDiskUse: true }
     );

### 6. Find the number of users who are absent and task is not submitted  between 15 oct-2020 and 31-oct-2020:

    ```bash
    db.getCollection('attendence').aggregate(
    [
    {
      $lookup: {
        from: 'topics',
        localField: 'topicId',
        foreignField: 'topicId',
        as: 'Absent'
      }
    },
    {
      $lookup: {
        from: 'tasks',
        localField: 'userId',
        foreignField: 'userId',
        as: 'Task-notSubmitted'
      }
    },
    { $unwind: '$Task-notSubmitted' },
    { $unwind: '$Absent' },
    {
      $match: {
        attended: false,
        'Absent.topicDate': {
          $gte: '2020-10-15',
          $lte: '2020-10-31'
        }
      }
    },
    {
      $match: {
        'Task-notSubmitted.dueDate': {
          $gte: '2020-10-15',
          $lte: '2020-10-31'
        },
        'Task-notSubmitted.submitted': false
      }
    },
    { $project: { _id: 0, 'Absent._id': 0 } },
    { $project: { 'task-notSubmitted._id': 0 } }
    ],
    { maxTimeMS: 60000, allowDiskUse: true }
    );
