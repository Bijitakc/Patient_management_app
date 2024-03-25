<script>
    import axios from "axios";

    let givenName = '';
    let familyName = '';
    let gender = '';
    let birthDateDay = '';
    let birthDateMonth = '';
    let birthDateYear = '';
    let phoneNumber = '';
    let jsonData = {
        resourceType: 'Patient',
        name: [],
        telecom: []
    };
    let createResponse = {};

    const handleSubmit = async () => {
        console.log("Form submitted.");
        createResponse = {};
        if (givenName === '' && familyName === '') {
            delete jsonData.name;
        }
        if (givenName !== '') {
            jsonData.name.push({given: givenName.split(" ")});
        }
        if (familyName !== '') {
            jsonData.name.push({family: familyName});
        }
        if (gender !== '') {
            jsonData.gender = gender;
        }
        if (birthDateYear !== ''){
            let fixedYear = ('000' + birthDateYear).slice(-4);
            jsonData.birthDate = fixedYear
            if (birthDateMonth !== ''){
                let fixedMonth = ('0' + birthDateMonth).slice(-2);
                jsonData.birthDate = `${fixedYear}-${fixedMonth}`
                if (birthDateDay !== ''){
                    let fixedDay = ('0' + birthDateDay).slice(-2);
                    jsonData.birthDate = `${fixedYear}-${fixedMonth}-${fixedDay}`
                }
            }
        }
        if (phoneNumber !== '') {
            jsonData.telecom.push({system: 'phone'});
            jsonData.telecom[0].value = phoneNumber;
        } else {
            delete jsonData.telecom
        }

        // Sending data
        try {
            const response = await axios.post(
                'https://lforms-fhir.nlm.nih.gov/baseR4/Patient', 
                jsonData, {headers: {'Content-Type': 'application/json'}}
            );
            console.log(response.status)
            if (response.status === 201) {
                createResponse = response.data
                createResponse.status = "201 created"
            }
            
        } catch (error) {
            console.error(error, 'error occured while creating a new patient.');
            createResponse = error.response.data
            createResponse.status = `${error.response.status}`
        }
        resetForm();
    }

    const resetForm = () => {
        givenName = '';
        familyName = '';
        gender = '';
        birthDateDay = '';
        birthDateMonth = '';
        birthDateYear = '';
        phoneNumber = '';
        jsonData = {
            resourceType: 'Patient',
            name: [],
            telecom: []
        };
    }

</script>

<div class="container">
    <div class="header">
        <h1>Patient App</h1>
    </div>
    <div class="operation-button">
        <button class="get-button" >
            <a href="/">Back to homepage</a>
        </button>
        <button class="get-button" >
            <a href="/list">Get All Patients</a>
        </button>
    </div>
    <div class="content-header">
        <h2>Create a new patient</h2>
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