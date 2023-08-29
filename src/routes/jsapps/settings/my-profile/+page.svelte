<script>
	import { Input, Label, Button, Select } from "flowbite-svelte";
	import { MultiSelect } from 'flowbite-svelte';
	import { onMount } from 'svelte';
	import "intl-tel-input/build/css/intlTelInput.css";
    import  intlTelInput from "intl-tel-input";

	// let profilepic ;
	let profilePicUrl;
	let id;
	let fullname = "";
	let email = "";
	let workPhone = "";
	let mobile = "";
	let timeZone = "";
	let reactiveTimezoneOptions = [];
	let address = "";
	let job;
	let languages = [];
  let selectedLanguages = []; // Initialize with default values
  let language = [];
	let iti;
	let iti1;
	let errors = {};
	let showValidation;
	let calendarColor;
	let file;
	let countryOptions;
	export let appData;
	console.log("appdata", appData);
	async function fetchData()  {
	  
	  const userDataResponse = await fetch("https://api.recruitly.io/api/me/profile?apiKey=TEST45684CB2A93F41FC40869DC739BD4D126D77" );
	  const userData = await userDataResponse.json();
	  console.log("userData",userData);
  
	  
	  id = userData.id;
	  profilePicUrl = userData.profilePicUrl;
	  fullname = userData.fullName;
	  email = userData.email;
	  workPhone = userData.workPhone;
	  mobile = userData.mobile;
	  timeZone = userData.timeZone;
	  job = userData.headline;
	  address = {        
		addressLine: userData.address.addressLine,
        cityName: userData.address.cityName,
        regionName: userData.address.regionName,
        postCode: userData.address.postCode,
        countryCode: userData.address.countryCode,
        latitude: userData.address.langitude,
        longitude: userData.address.longitude,		
	  }
	  selectedLanguages = userData.languages,
	  calendarColor = userData.calendarColor
,
	  console.log(workPhone,"languages")
	  
	};
  
	async function updateData() {
		workPhone = iti.getNumber();
		mobile = iti1.getNumber();
	  const requestData = {
		profilePicUrl,
		fullName:fullname,
		email,
        workPhone,
		address,
		calendarColor,
		headline:job,
		mobile,
		timeZone,
		languages:selectedLanguages,
		};


	  console.log(JSON.stringify(requestData));


	  try {
		const response = await fetch("https://api.recruitly.io/api/me/save?apiKey=TEST45684CB2A93F41FC40869DC739BD4D126D77",
		{
		  method: "POST",
		  headers: {
			"Content-Type": "application/json",
			
		  },
		  body: JSON.stringify(requestData),
		});
		if (!response.ok) {
      throw new Error('Network response was not ok');
    }

    const updatedData = await response.json();
	console.log(response);
    console.log('Updated data from API:', updatedData);
  	} catch (error) {
    console.error('Error updating data:', error);

  	}
	}

	async function fetchTimeZone() {
        try {
            const response = await fetch(
		"https://api.recruitly.io/api/lookup/timezone?apiKey=TEST45684CB2A93F41FC40869DC739BD4D126D77"
	)
		
    const responseData = await response.json();

	if (Array.isArray(responseData.data)) {
    // Map the response data to the format required for options
    	reactiveTimezoneOptions = responseData.data.map((timeZone) => ({
        	value: timeZone.code,
        	name: timeZone.name,
    	}));
    	console.log("API response:", responseData);
	} else {
    	console.error("Invalid currency data format:", responseData);
	}
	} catch (error) {
	console.error("Error fetching currency options:", error);
	}

	}

	onMount(async() => {
	  fetchTimeZone();
	  fetchCountryData();
	  await fetchData();
	 await fetchLanguages();
	 
	  const inputElement = document.querySelector("#phone-input");

		iti = intlTelInput(inputElement, {
		utilsScript:
			"https://cdn.jsdelivr.net/npm/intl-tel-input@16.0.3/build/js/utils.js",
		nationalMode: true,
		initialCountry: "gb",
		preferredCountries: ["gb", "in"],
		});
		const inputElement1 = document.querySelector("#mobile-input");

        iti1 = intlTelInput(inputElement1, {
            utilsScript:
                "https://cdn.jsdelivr.net/npm/intl-tel-input@16.0.3/build/js/utils.js",
            nationalMode: true,
            initialCountry: "gb",
            preferredCountries: ["af", "in"],
        });
		iti.setNumber(workPhone);
	    iti1.setNumber(mobile);
	});

	async function fetchLanguages() {
    try {
      const response = await fetch(
        "https://api.recruitly.io/api/lookup/languages?apiKey=TEST45684CB2A93F41FC40869DC739BD4D126D77" 
      );
      const responseData = await response.json();

      if (Array.isArray(responseData.data)) {
        languages = responseData.data.map((lang) => ({
          name: lang.name,
          value: lang.code,
        }));
        
        // Filter languages based on preselected values
        language = languages.filter((lang) =>
          selectedLanguages.includes(lang.value)
						// console.log(language,"lang")
        );
      } else {
        console.error('Invalid API response format:', responseData);
      }
    } catch (error) {
      console.error('Error fetching data:', error);
    }
  }

  function handleSelection(event) {
    if (event && event.detail && event.detail.value) {
      selectedLanguages = event.detail.value;
      language = languages.filter((lang) =>
        selectedLanguages.includes(lang.value)
      );
    }
  }
		
    
	

	function validateForm() {
        errors = {};

        if (!fullname || fullname.trim() === "") {
            errors.fullname = "Full Name is required";
        }

        if (!address.cityName || !address.cityName.trim() === "") {
            errors.cityName = "City Name is required";
        }

		if (!address.countryCode || address.countryCode.trim() === "") {
            errors.countryCode = "Country Name is required";
        }

        if (!workPhone || !workPhone.trim() === "") {
        	 errors.workPhone = "Work Phone is required";
        } 
		
        if (job.trim() === "" || !job) {
            errors.job = "Job/Headline is required";
        }

        return errors;
    }
	async function handleSubmit() {
        showValidation = true;
        const errors = validateForm();

        if (Object.keys(errors).length > 0) {
            return;
        }

        await updateData();
    }
	const handleLogoChange = (event) => {
        file = event.target.files[0];
        if (file) {
            const reader = new FileReader();

            reader.onload = () => {
                profilePicUrl = reader.result;

                showLogoButton();
            };

            reader.readAsDataURL(file);
        }
    };
    const showLogoButton = () => {
        const inputElement = document.getElementById("logoInput");
        const logoImage = document.getElementById("logoImage");

        if (file || profilePicUrl) {
            logoImage.style.display = "block";
            inputElement.style.display = "none";
        } else {
            logoImage.style.display = "none";
            inputElement.style.display = "block";
        }
    };

    const uploadImage = () => {
        if (!file) {
            console.log("No file Selected.");
            return;
        }

        const formData = new FormData();
        formData.append("file", file);
        formData.append("profilePic", "true");
        formData.append("type", "USER");

        fetch(
            "https://api.recruitly.io/api/image/upload?apiKey=TEST45684CB2A93F41FC40869DC739BD4D126D77",
            {
                method: "POST",
                body: formData,
            }
        )
            .then((response) => {
				
                if (!response.ok) {
                    throw new Error("Network response was not ok");
                }
                return response.json();
            })
            .then((data) => {
                console.log(data);
            })
            .catch((error) => {
                console.error("Error during image upload:", error);
            });
    };

	const fetchCountryData = async () => {
        try {
            const response = await fetch(
                "https://api.recruitly.io/api/lookup/countries?apiKey=TEST45684CB2A93F41FC40869DC739BD4D126D77"
            );
            const responseData = await response.json();

            if (Array.isArray(responseData.data)) {
                // Map the response data to the format required for options
                countryOptions = responseData.data.map((country) => ({
                    value: country.code,
                    name: country.name,
                }));

                console.log("Country API response:", responseData);
            } else {
                console.error("Invalid country data format:", responseData);
            }
        } catch (error) {
            console.error("Error fetching country options:", error);
        }
    };

  </script>
  
  	<main class="container">
		<div class="form-container">
	  	<form on:submit|preventDefault>
		
			<div class="mb-6">
				<Label for="firstname" class="mb-2"><b>About me</b></Label>
				<hr>
			</div>
		
			<div
			class="mb-3 business-logo-container"
			style="padding-top: 20px"
			on:change={uploadImage}
		>
		<Label for="profile" class="mb-2">Profile Picture</Label>

		<input
			type="file"
			accept="image/*"
			id="logoInput"
			class="file-input"
			on:change={handleLogoChange}
		/>

		<!-- svelte-ignore a11y-click-events-have-key-events -->
		
			<img
				alt
				src={profilePicUrl}
				id="logoImage"
				on:click={() => {
				document.getElementById("logoInput").click();
				}}
			/>
		</div>
		<div class="mb-6">
		  <Label for="firstname" class="mb-2">Full Name<span class="required">*</span></Label>
		  <Input
			type="text"
			id="fullname"
			bind:value={fullname}
			placeholder="Full Name" />
			{#if showValidation && errors.fullname}
                <p class="error">{errors.fullname}</p>
            {/if}
		</div>
		<div class="mb-6">
		  <Label for="email" class="mb-2">Email<span class="required">*</span></Label>
		  <Input
			type="text"
			id="email"
			bind:value={email}
			placeholder="Email"
			readonly />
		</div>
		<div class="mb-6">
			<Label for="phone" class="mb-2">Work Phone<span class="required">*</span></Label>
			<Input
			  type="tel"
			  id="phone-input"
			  bind:value={workPhone}
			  style="width:800px ;" 
			  required />
			  {#if showValidation && errors.workPhone}
                <p class="error">{errors.workPhone}</p>
            {/if}
		</div>
		<div class="mb-6">
			<Label for="timeZone" class="mb-2">Time Zone<span class="required">*</span></Label>
			<Select class="mt-2" items={reactiveTimezoneOptions} bind:value={timeZone} />		   
		</div>
		<div class="mb-6">
			<Label for="address" class="mb-2">City Name<span class="required">*</span></Label>
			<Input
			  type="text"
			  id="address"
			  bind:value={address.cityName}
			  placeholder="City Name" />
			  {#if showValidation && errors.cityName}
                <p class="error">{errors.cityName}</p>
            {/if}
		</div>
		<div class="mb-6">
            <Label for="country" class="mb-2">Country<span class="required">*</span></Label>
            <Select
                class="mt-2"
                items={countryOptions}
                bind:value={address.countryCode}
                />
                {#if showValidation && errors.countryCode}
                <p class="error">{errors.countryCode}</p>
            {/if}
            
        </div>
		<div class="mb-6">
			<Label for="job" class="mb-2">Job Title</Label>
			<Input
			  type="text"
			  id="job"
			  bind:value={job}
			  placeholder="Job Title" />
			  {#if showValidation && errors.job}
                <p class="error">{errors.job}</p>
            {/if}
		</div>
		<div class="mb-6">
			<Label for="phone" class="mb-2">Mobile Phone</Label>
			<Input
			  type="tel"
			  id="mobile-input"
			  bind:value={mobile}
			  style="width:800px ;"/>
		</div>
		<div class="mb-6">
			<Label class="mb-2"> Languages </Label>
			{#if languages.length > 0}
			  <MultiSelect
				items={languages}
				bind:value={selectedLanguages}
				on:change={handleSelection}
			  />
			{/if}
		</div>	
		<div class="mb-6">
			<Label for="color" class="mb-2">Calendar Color</Label>
			<Input
			  type="color"
			  id="color"
			  bind:value={calendarColor}
			  style="width: 800px; height: 40px;"/>
		</div>
		<Button
		  class="btn btn-primary"
		  type="button"
		  on:click={handleSubmit}
		  style="background-color: indigo;">
		  Update Profile
		</Button>
	  </form>
	</div>
  </main>
  
  <style>
	.form-container{
        width: 800px;
		margin-top: 10px;
	}
	.profile-pic {
		width: 150px; 
    	height: 150px; 
        max-width: 20%;
		margin-top: 10px;
    	border-radius: 50%;
    	overflow: hidden;
		margin-left: 300px;
    }
	.container {
    display: grid;
    place-items: center;
    height: 100vh;
	}

	.error {
        color: rgb(255, 162, 0);
        font-size: 14px;
        margin-top: 4px;
    }

	.required {
        color: rgb(199, 27, 27);
        margin-left: 2px;
    }
	.business-logo-container {
        position: relative;
    }

    .business-logo-container input[type="file"] {
        display: none;
    }

    .business-logo-container img {
        cursor: pointer;
        display: block; /* Ensure the image is displayed by default */
    }

    #logoImage {
		display: flex;
    justify-content: center;
    align-items: center;
    width: 150px;
    height: 150px;
    border-radius: 50%;
    overflow: hidden;
    margin: 0 auto;
    }
  </style>
  