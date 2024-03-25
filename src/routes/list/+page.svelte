<script>
    import axios from "axios";
    import { onMount } from "svelte";

    let patients = [];
    let nextPageUrl = '';
    let previousPageUrl = '';

    onMount(() => {
        fetchData();
    });

    const fetchData = async () => {
        try {
            const response = await axios.get('https://lforms-fhir.nlm.nih.gov/baseR4/Patient?_count=100');
            patients = response.data.entry;

            // Extracting the next page URL from response link
            const nextLink = response.data.link.find(link => link.relation === 'next');
            nextPageUrl = nextLink ? nextLink.url : '';
        } catch (error) {
            console.error(error, 'error occured while fetching patients');
        }
    };

    const fetchNextPage = async () => {
        try {
            const response = await axios.get(nextPageUrl);
            patients = response.data.entry;

            // Extracting the next page URL from response link
            const nextLink = response.data.link.find(link => link.relation === 'next');
            nextPageUrl = nextLink ? nextLink.url : '';
            const prevLink = response.data.link.find(link => link.relation === 'previous');
            previousPageUrl = prevLink ? prevLink.url : '';
        } catch (error) {
            console.error(error, 'error occured while fetching patients');
        }
        const contentHeader = document.querySelector('.content-header');
        if (contentHeader) {
            contentHeader.scrollIntoView({ behavior: 'smooth' });
        }
    };

    const fetchPreviousPage = async () => {
        try {
            const response = await axios.get(previousPageUrl);
            patients = response.data.entry;

            // Extracting the next page URL from response link
            const nextLink = response.data.link.find(link => link.relation === 'next');
            nextPageUrl = nextLink ? nextLink.url : '';
            const prevLink = response.data.link.find(link => link.relation === 'previous');
            previousPageUrl = prevLink ? prevLink.url : '';
        } catch (error) {
            console.error(error, 'error occured while fetching patients');
        }
        const contentHeader = document.querySelector('.content-header');
        if (contentHeader) {
            contentHeader.scrollIntoView({ behavior: 'smooth' });
        }
    };

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
            <a href="/create">Create New Patient</a>
        </button>
    </div>
    {#if patients.length !== 0}
        <div class="content-header">
            <h2>Patients List</h2>
        </div>
        <div class="content">
            <table>
                <thead>
                    <tr>
                        <th>Given Name</th>
                        <th>Family Name</th>
                        <th>Gender</th>
                        <th>Birth Date</th>
                        <th>Edit</th>
                    </tr>
                </thead>
                <tbody>
                    {#each patients as {resource: patient}}
                        <tr>
                            <td>
                                {#if patient.name}
                                    {#if patient.name[0].given}
                                        {patient.name[0].given.join(' ')}
                                    {/if}
                                {/if}
                            </td>
                            <td>
                                {#if patient.name}
                                    {#if patient.name[0].family}
                                        {patient.name[0].family}
                                    {/if}
                                {/if}
                            </td>
                            <td>
                                {#if patient.gender}
                                    {patient.gender}
                                {/if}
                            </td>
                            <td>
                                {#if patient.birthDate}
                                    {patient.birthDate}
                                {/if}
                            </td> 
                            <td>
                                <button class="edit-button">
                                    <a href={`/edit/${patient.id}`}>
                                        Edit
                                    </a>
                                </button>
                            </td>                       
                        </tr>
                    {/each}
                </tbody>
            </table>
        </div>
    {/if}
    <div class="paginationElements">
        {#if previousPageUrl}
            <button class="newPageButton" on:click={() => fetchPreviousPage()}>
                Prev
            </button>
        {/if}
        {#if nextPageUrl}
            <button class="newPageButton" on:click={() => fetchNextPage()}>
                Next
            </button>
        {/if}
    </div>
</div>

<style>
    a {
        text-decoration: none;
        color: inherit;
    }
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
    .newPageButton {
        font-size: large;
        padding: 7px 30px;;
        margin-bottom: 20px;
        background-color: #9fc381cc;
        border: none;
    }
    table {
        width: 100%;
        border-collapse: collapse;
        margin-bottom: 20px;
    }
    th, td {
        border: 1px solid black;
        padding: 8px;
        text-align: left;
    }
    th {
        background-color: #bbd8a8;
    }
    tbody {
        background-color: #e3f1d9;
    }
</style>