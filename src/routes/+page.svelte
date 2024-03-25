<script>
    import axios from "axios";

    let searchInput = '';
    let searchResults = {};

    const isPhoneNumber = (searchInput) => {
        return /^\d+$/.test(searchInput);
    }
    const searchSubmit= async () => {
        console.log("Search submitted.", searchInput)
        let searchParam = ''
        console.log (isPhoneNumber(searchInput))
        if (searchInput){
            if (isPhoneNumber(searchInput)) {
                searchParam = `phone=${searchInput}`
            } else {
                searchParam = `name=${searchInput}`
            }
            try {
                const response =  await axios.get(`https://lforms-fhir.nlm.nih.gov/baseR4/Patient?${searchParam}`)
                searchResults = response.data
                console.log("okay", response.data)
            } catch (error) {
                console.error(error, 'error occured while creating a new patient.');
            }
        }
    }
</script>

<div class="container">
    <div class="header">
        <h1>Patient App</h1>
    </div>
    <div class="search-container">
        <input type="text" id="searchInput" placeholder="Search..." bind:value={searchInput} >
        <button class="search-button" on:click={() => searchSubmit()}>
            Search
        </button>
    </div>
    <div class="operation-button">
        <button class="get-button" >
            <a href="/list">Get All Patients</a>
        </button>
        <button class="get-button" >
            <a href="/create">Create New Patient</a>
        </button>
    </div>
    <div class="returnedResponse">
        {#if searchResults}
            <pre> {JSON.stringify(searchResults, null, 2)}</pre>
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
    .search-container {
        display: flex;
        align-items: center;
        margin: 10px 10px;
    }
    #searchInput {
        padding: 10px;
        border: 1px solid #ccc;
    }

    .search-button {
        padding: 10px 20px;
        font-size: 14px;
        background-color: #4CAF50; /* Green */
        color: white;
        border: none;
        cursor: pointer;
    }

    .search-button:hover {
        background-color: #45a049; /* Darker green */
    }

</style>