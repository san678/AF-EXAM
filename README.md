# AF-EXAM
Application Framewwork sample exam crud


simple mern crud application

create backend folder
in backend folder
npm init
install packages - npm i mongoose express dotenv cors
create folders models and routes
create file server.js
change pachkage.json start name
create file named as .env
npm i nodemon "dev": "nodemon server.js"

-------------------------------------------------------------------------------------------

in main folder npx create-react-app frontend
in frontend folder
delete setupTest.js, reportWeb.js, logo.svg, index.css, App.test.js
create folder components in src folder
copy bootstrap links to index.html
create file AddStudent.js in component folder
npm install react-router-dom
npm install axios for handle http requests
create file UpdateStudent.js

-----------------------------------------------------------------------------------------------------
backend


server.js


const express = require('express') // import express
const mongoose = require('mongoose') // import mongoose
const cors = require('cors') // import cors
require('dotenv').config() // import dotenv
const studentRoutes = require('./routes/students.js') // import routes

const app = express() // create express app
const PORT = process.env.port || 7070; // set port

app.use(cors()) // use cors
app.use(express.json()) // use express json

// connect to mongodb
mongoose.connect(process.env.MONGODB_URL)
.then(() => console.log('Connected to MongoDB'))
.catch(err => console.log(err))

app.use("/student", studentRoutes);

app.listen(PORT, ()=> console.log(`server is up and running on port : ${PORT}`))

backend -> models -> Student.js
const mongoose = require('mongoose');

const studentSchema = new mongoose.Schema({
    name:{
        type: String,
        required: true,
    },
    age:{
        type: Number,
        required: true,
    },
    gender: {
        type: String,
        required: true,
    }
})

module.exports = mongoose.model('Student', studentSchema);// Path: MERN-crud\Backend\models\Student.js


backend -> routes -> students.js
const router = require('express').Router(); // import express router
let Student = require('../models/Student'); // import student model

// create a new student
router.route('/add').post((req, res) => {

    const name = req.body.name;
    const age = Number(req.body.age);
    const gender = req.body.gender;

    const newStudent = new Student({
        name,
        age,
        gender
    })

    newStudent.save().then(()=>{
        res.json("Student added")
    }).catch((err)=>{
        console.log(err);
    })
})

// get all students
router.route('/').get((req, res)=>{
    Student.find().then((students)=>{
        res.json(students)
    }).catch((err)=>{
        console.log(err);
    })
})

// update a student
router.route('/update/:id').put(async(req, res)=>{
    let userId = req.params.id;
    const {name,age, gender} = req.body; // updated detail from frontend

    const updateStudent = {
        name,
        age,
        gender
    }

    const update = await Student.findByIdAndUpdate(userId, updateStudent)
    .then(()=>{
    res.status(200).send({status: "User updated"})
    }).catch((err)=>{
        console.log(err);
        res.status(500).send({status: "Error with updating data", error: err.message});
    })
})

// delete a student
router.route('/delete/:id').delete(async(req,res)=>{
    let userId = req.params.id;

    await Student.findByIdAndDelete(userId)
    .then(()=>{
        res.status(200).send({status: "User deleted"});
    }).catch((err)=>{
        console.log(err);
        res.status(500).send({status: "Error with deleting data", error: err.message});
    })
})

// get one student
router.route('/get/:id').get(async(req, res)=>{
    let userId = req.params.id;

    const user = await Student.findById(userId)
    .then((student)=>{
        res.status(200).send({status: "User fetched", student})
    }).catch((err)=>{
        console.log(err.message);
        res.status(500).send({status: "Error with get user", error: err.message});
    })
})

module.exports = router;
----------------------------------------------------------------------------------------------------------------------------

frontend
App.js


import './App.css';
import Header from './components/Header';
import {BrowserRouter as Router, Route, Routes} from 'react-router-dom'

import AddStudent from './components/AddStudent';
import ViewAllStudents from './components/ViewAllStudents';
import UpdateStudent from './components/UpdateStudent';

function App() {
  return (
    <div>
    <Router>
        <Header />
        <Routes>
          <Route exact path="/add" element={<AddStudent/>}/>
          <Route exact path="/view" element={<ViewAllStudents/>}/>
          <Route exact path="/update/:id" element={<UpdateStudent/>}/>
        </Routes>
    </Router>
    </div>
       
  );
}

export default App;

AddStudent.js
import React, { useState} from 'react';
import axios from 'axios';
import { useNavigate } from 'react-router-dom';
function AddStudent(){

    const [name, setName] = useState("");
    const [age, setAge] = useState("");
    const [gender, setGender] = useState("");

    let navigate = useNavigate();

    //submit button function
    function submitButton(e){
      e.preventDefault();  //prevent default behaviour of form submit
      //alert("Student added");

      const newStudent = {
        name,
        age,
        gender
      }
     
      //backend url: http://localhost:7070/student/add
      axios.post("http://localhost:7070/student/add", newStudent)
      .then(()=>{
        console.log(newStudent);

        setName("");
        setAge("");
        setGender("");

        alert("Student added");
       
        navigate("/view");

      }).catch((err)=>{
        alert(err);
      })

    }

    return(
      <div className="container">

      <form onSubmit={submitButton}>
        <div className="mb-3">
          <label for="name" className="form-label">Student name</label>
          <input type="text" className="form-control" id="name" aria-describedby="name"
          onChange={(e)=>{
            setName(e.target.value);
          }}
         
          />
        </div>
       
        <div className="mb-3">
          <label for="age" className="form-label">Student age</label>
          <input type="text" className="form-control" id="age" aria-describedby="age"
          onChange={(e)=>{
            setAge(e.target.value);
          }}/>
        </div>

        <div className="mb-3">
          <label for="gender" className="form-label">Student gender</label>
          <input type="text" className="form-control" id="gender" aria-describedby="gender"
          onChange={(e)=>{
            setGender(e.target.value);
          }}/>
        </div>

        <button type="submit" className="btn btn-primary">Submit</button>
      </form>

      </div>
     
       
    )
}

export default AddStudent;

UpdateStudent.js
import React from 'react';
import {useState, useEffect} from 'react'; //useEffect is used to fetch data from backend
import axios from 'axios'; //axios is used to make http requests to backend
import { useNavigate, useParams } from 'react-router-dom'; //useNavigate is used to navigate to a different page

function UpdateStudent(){

    const [name, setName] = useState("");
    const [age, setAge] = useState("");
    const [gender, setGender] = useState("");

    const paramID = useParams("");
    let navigate = useNavigate();

    useEffect(() => {
       
          axios
            .get(`http://localhost:7070/student/get/`+ paramID.id) //backend url: http://localhost:7070/student/
            .then((res) => {
              console.log(res.data);
              setName(res.data.student.name);
                setAge(res.data.student.age);
                setGender(res.data.student.gender);
              //console.log(setStudents);
            })
            .catch((err) => {
              alert(err);
              console.log(err);
            });
      }, []);

    function submitButton(e){
        e.preventDefault();  //prevent default behaviour of form submit
        //alert("Student added");
 
        const newStudent = {
          name,
          age,
          gender
        }
       
        //backend url: http://localhost:7070/student/add
        axios.put(`http://localhost:7070/student/update/${paramID.id}`, newStudent)
        .then(()=>{
          console.log(newStudent)
          alert("Student updated");
          setName("");
          setAge("");
          setGender("");

          navigate("/view");
 
        }).catch((err)=>{
          alert(err);
        })
 
      }



    return(

        <div>
            <form onSubmit={submitButton}>
        <div className="mb-3">
          <label for="name" className="form-label">Student name</label>
          <input type="text" className="form-control" id="name" aria-describedby="name"
          onChange={(e)=>{
            setName(e.target.value);
          }}
          value={name}/>
        </div>
       
        <div className="mb-3">
          <label for="age" className="form-label">Student age</label>
          <input type="text" className="form-control" id="age" aria-describedby="age"
          onChange={(e)=>{
            setAge(e.target.value);
          }}
          value={age}/>
        </div>

        <div className="mb-3">
          <label for="gender" className="form-label">Student gender</label>
          <input type="text" className="form-control" id="gender" aria-describedby="gender"
          onChange={(e)=>{
            setGender(e.target.value);
          }}
          value={gender}/>
        </div>

        <button type="submit" className="btn btn-primary">Update</button>
      </form>
        </div>
    )
}

export default UpdateStudent;

ViewAllStudent.js
import React from "react";
import { useState, useEffect } from "react"; //useEffect is used to fetch data from backend
import axios from "axios"; //axios is used to make http requests to backend
import { useNavigate, useParams } from "react-router-dom"; //useNavigate is used to navigate to a different page
import { Link } from "react-router-dom";


function ViewAllStudents() {
  const [students, setStudents] = useState([]); //students is an array of objects [{}

  const [search, setSearch] = useState(""); //search is a string
   //const id = useParams(); //get the id from the url
  useEffect(() => {
    function getStudents() {
      axios
        .get(`http://localhost:7070/student/`) //backend url: http://localhost:7070/student/
        .then((res) => {
          console.log(res.data);
          setStudents(res.data);
          //console.log(setStudents);
        })
        .catch((err) => {
          alert(err);
          console.log(err);
        });
    }
    getStudents();
  }, []);

    let navigate = useNavigate();
    function gotoEdit(id) {
        navigate("/update/" + id);
    }


  function DeleteStudent(id) {


    axios
        .delete(`http://localhost:7070/student/delete/${id}`) //backend url: http://localhost:7070/student/delete/${userID}
        .then((res) => {
            console.log(res.data);
            alert("Student deleted");
            //navigate("/view");
        })
        .catch((err) => {
            alert(err);
            console.log(err);
        });
    }

  return (
    <div class="row">
      {students.map((item, key) => (
        <div class="col-sm-6 mb-3 mb-sm-0" key={key}>
          <div class="card">
            <div class="card-body">
              <h5 class="card-title">{item._id}</h5>
              <p class="card-text">Name : {item.name}</p>
              <p class="card-text">Age : {item.age}</p>
              <p class="card-text">Gender : {item.gender}</p>

              <div class="btn-group" role="group" aria-label="First group">
              <div class="btn btn-warning"
              onClick={()=>{gotoEdit(item._id)}} > Edit
                {/* <Link to={"/update/" + item._id}>
               
                  Edit
             
              </Link> */}

                </div>
                <a href="/view" class="btn btn-danger" onClick={()=> DeleteStudent(item._id)}>
                  Delete
                </a>
              </div>
            </div>
          </div>
        </div>
      ))}
    </div>
  );
}

export default ViewAllStudents;

AddStudent.js without materialui
import React, { useState } from 'react';
import axios from 'axios';
import { useNavigate } from 'react-router-dom';
import 'bootstrap/dist/css/bootstrap.min.css';  // Importing Bootstrap CSS

function AddStudent() {
  const [name, setName] = useState("");
  const [age, setAge] = useState("");
  const [gender, setGender] = useState("");

  let navigate = useNavigate();

  // submit button function
  const submitButton = (e) => {
    e.preventDefault(); // prevent default behavior of form submit

    const newStudent = {
      name,
      age,
      gender,
    };

    // backend URL: http://localhost:7070/student/add
    axios.post("http://localhost:7070/student/add", newStudent)
      .then(() => {
        console.log(newStudent);

        setName("");
        setAge("");
        setGender("");

        alert("Student added");
        navigate("/view");
      })
      .catch((err) => {
        alert(err);
      });
  };

  return (
    <div className="container mt-5">
      <h2>Add New Student</h2>
      <form onSubmit={submitButton}>
        <div className="mb-3">
          <label htmlFor="name" className="form-label">Student Name</label>
          <input
            type="text"
            className="form-control"
            id="name"
            value={name}
            onChange={(e) => setName(e.target.value)}
            required
          />
        </div>

        <div className="mb-3">
          <label htmlFor="age" className="form-label">Student Age</label>
          <input
            type="number"
            className="form-control"
            id="age"
            value={age}
            onChange={(e) => setAge(e.target.value)}
            required
          />
        </div>

        <div className="mb-3">
          <label htmlFor="gender" className="form-label">Student Gender</label>
          <input
            type="text"
            className="form-control"
            id="gender"
            value={gender}
            onChange={(e) => setGender(e.target.value)}
            required
          />
        </div>

        <button type="submit" className="btn btn-primary">Submit</button>
      </form>
    </div>
  );
}

export default AddStudent;

CSS part
/* Add this CSS to your App.css or a dedicated CSS file */

.container {

  max-width: 600px;

  margin: auto;

}



h2 {

  margin-bottom: 20px;

}



.btn-primary {

  width: 100%;

}





