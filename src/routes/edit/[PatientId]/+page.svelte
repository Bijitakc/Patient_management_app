<script>
    import { onMount } from "svelte";
    import { page } from '$app/stores';
    import axios from "axios";

    let patientId;
    let givenName = '';
    let familyName = '';
    let gender = '';
    let birthDateDay = '';
    let birthDateMonth = '';
    let birthDateYear = '';
    let phoneNumber = '';
    let patient = {};
    let createResponse = {};
    let given = [];

    $: {
        patientId = $page.params.PatientId;
    }

    const fetchData = async () => {
        const response = await axios.get(`https://lforms-fhir.nlm.nih.gov/baseR4/Patient/${patientId}`)
        patient = response.data
        if (patient.name && patient.name[0].given) {
            given = patient.name[0].given;
            givenName = given.join(' ');
        }
        if (patient.name && patient.name[0].family) {
            familyName = patient.name[0].family;
        }
        if (patient.gender) {
            gender = patient.gender;
        }
        if (patient.birthDate) {
            console.log("", )
            let birthDateList = patient.birthDate.split("-")
            if (birthDateList.length === 3) {
                birthDateYear = birthDateList[0]
                birthDateMonth = birthDateList[1]
                birthDateDay = birthDateList[2]
            } else if (birthDateList.length === 2) {
                birthDateYear = birthDateList[0]
                birthDateMonth = birthDateList[1]
            } else if (birthDateList.length === 1) {
                birthDateYear = birthDateList[0]
            }
        }
        if (patient.telecom){
            if (patient.telecom[0].system === 'phone' && patient.telecom[0].value) {
                phoneNumber = patient.telecom[0].value
            }    
        }
    }

    const handleSubmit = async () => {
        console.log("Form submitted.");

        // Validation
        if (birthDateDay !== '' && birthDateMonth === '') {
            alert('Please provide the birth month or remove birthdate day.');
            return;
        }
        
        createResponse = {};
        if (givenName !== '') {
            if (patient.name) {
                patient.name[0].given = givenName.split(" ");
            } else {
                patient.name.push({given: givenName.split(" ")});
            }
        }
        if (familyName !== '') {
            if (patient.name) {
                patient.name[0].family = familyName;
            } else {
                patient.name.push({family: familyName});
            }
        }
        if (gender !== '') {
            patient.gender = gender;
        }
        if (birthDateYear !== ''){
            let fixedYear = ('000' + birthDateYear).slice(-4);
            patient.birthDate = fixedYear
            if (birthDateMonth !== ''){
                let fixedMonth = ('0' + birthDateMonth).slice(-2);
                patient.birthDate = `${fixedYear}-${fixedMonth}`
                if (birthDateDay !== ''){
                    let fixedDay = ('0' + birthDateDay).slice(-2);
                    patient.birthDate = `${fixedYear}-${fixedMonth}-${fixedDay}`
                }
            }
        }
        if (phoneNumber !== '') {
            if (patient.telecom){
                console.log("Yes")
                patient.telecom[0].system = 'phone';
                patient.telecom[0].value = phoneNumber;
            } else {
                patient.telecom = []
                patient.telecom.push({system: 'phone'})
                patient.telecom[0].value = phoneNumber
            }
        } else {
            delete patient.telecom
        }

        // Updating the patient resource
        try {
            const response = await axios.put(
                `https://lforms-fhir.nlm.nih.gov/baseR4/Patient/${patientId}`, 
                patient, {headers: {'Content-Type': 'application/json'}}
            );
            if (response.status === 200) {
                createResponse = response.data
                createResponse.status = "200 updated"
            }
            
        } catch (error) {
            console.error(error, 'error occured while updating patient.');
            createResponse = error.response.data
            createResponse.status = `${error.response.status}`
        }

    }
    onMount((patientId) => {
        fetchData();
    })
</script>

<div class="container">
    <div class="header">
        <h1>Patient App</h1>
    </div>
    <div class="operation-button">
        <button class="get-button" >
            <a href="/list">Get All Patients</a>
        </button>
        <button class="get-button" >
            <a href="/create">Create New Patient</a>
        </button>
    </div>
    <div class="content-header">
        <h2>Updating patient {patientId}</h2>
    </div>
    <div class="patient-form">
        <form on:submit|preventDefault={handleSubmit}>
            <div class="form-line">
                <label for="givenName">Given Name:</label>
                <input type="text" id="givenName" bind:value={givenName} >
            </div>
          
            <div class="form-line">
                <label for="familyName">Family Name:</label>
                <input type="text" id="familyName" bind:value={familyName} >
            </div>
          
            <div class="form-line">
                <label for="gender">Gender</label>
                <select id="gender" bind:value={gender} >
                <option value="">Select Gender</option>
                <option value="male">Male</option>
                <option value="female">Female</option>
                <option value="other">Other</option>
                <option value="unknown">Unknown</option>
                </select>
            </div>
          
            <div class="form-line">
                <label for="birthDate">Date of Birth</label>
                <input type="number" id="birthDateDay" min="1" max="30" placeholder="dd" bind:value={birthDateDay} >
                <input type="number" id="birthDateMonth" min="1" max="12" placeholder="mm" bind:value={birthDateMonth} >
                <input type="number" id="birthDateYear" min="1" max="2024" placeholder="yyyy" bind:value={birthDateYear} >
            </div>

            <div class="form-line">
                <label for="phoneNumber">Phone Number</label>
                <input type="text" id="phoneNumber" bind:value={phoneNumber} >
            </div>
          
            <div class="form-line">
                <button class="submit" type="submit">Submit</button>
            </div>
          </form>
    </div>
    <div class="returnedResponse">
        {#if createResponse.status}
            <h2>Response</h2>
            <h4>Status: {createResponse.status}</h4>
            <pre> {JSON.stringify(createResponse, null, 2)}</pre>
        {/if}
    </div>
</div>

<style>
    .container {
        display: flex;
        flex-direction: column;
        height: 100%;
        padding-left: 30px;
    }
    .header {
        margin-bottom: 20px; 
        text-align: center;
    }
    .get-button {
        font-size: large;
        padding: 10px;;
        margin-bottom: 20px;
        background-color: #9fc381cc;
        border: none;
    }
    a {
        text-decoration: none;
        color: inherit;
    }
    .form-line {
        margin: 10px 10px;
    }
    .patient-form {
        padding-top: 10px;
        padding-left: 15px;
        margin-right: 20px;
        background-color: rgb(225, 225, 225);
    }
    #birthDateDay, #birthDateMonth {
        width: 35px;
    }
    #birthDateYear {
        width: 70px;
    }
    .submit {
        font-size: medium;
        padding: 5px 10px;;
        margin: 10px 0px;
        background-color: #428c24cc;
        border: none;
    }
    .returnedResponse {
        min-height: 200px;
        padding: 7px 10px;
        margin-right: 20px;
    }
</style>