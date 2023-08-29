<script>
  import { Input, Label, Button, Select } from "flowbite-svelte";
  import { onMount } from "svelte";

  import { MultiSelect } from "flowbite-svelte";
  import "intl-tel-input/build/css/intlTelInput.css";
  import intlTelInput from "intl-tel-input";

  export let file;
  export let iti;
  export let appData;
  export let id;
  export let name;
  // export let address;
  export let phone = "";
  export let website;
  export let currencyOptions;
  export let preferredDateformat;
  export let reactiveTimezoneOptions;
  export let currencyCode;
  export let timeZone;
  export let countryOptions = [];
  export let errors = {};
  export let showValidation;
  export let preferredCountryCode = [];
  export let preferredCountryCode1 = [];
  export let preferredCountries = [];
  export let cityName;
  export let postCode;
  export let addressLine;
  export let regionName;
  export let countryCode;
  export let latitude;
  export let longitude;
  export let logo;
  export let countries = [];
  export let selectedCodes = [];
  export let selectedLabels = [];

  let selectedCountry = null;

  let isValid = false;

  const fetchData = async () => {
      try {
          const response = await fetch(
              "https://api.recruitly.io/api/business/profile?apiKey=TEST45684CB2A93F41FC40869DC739BD4D126D77"
          );
          const data = await response.json();
          console.log(data, "data");

          logo = data.logo.url,
              id = data.id,
              name = data.name,
              
                  addressLine= data.address.addressLine,
                  cityName= data.address.cityName,
                  regionName= data.address.cityRegion,
                  postCode= data.address.postCode,
                  countryCode= data.address.countryCode,
                  latitude= data.address.latitude,
                  longitude= data.address.longitude,
              
              
          
          selectedCountry = data.address.countryCode;
          postCode = data.address.postCode;
          phone = data.phone;
          website = data.website,
              currencyCode = data.currencyCode.code,
              preferredDateformat = data.preferredDateFormat,
              timeZone = data.timeZone;
          preferredCountryCode = data.mobilePreferences.preferredCountryCode;
          preferredCountries = data.mobilePreferences.preferredCountries;

          selectedLabels = preferredCountries.split(",");
          preferredCountryCode1 = preferredCountryCode.toUpperCase();

          // iti.setNumber(phone);
      } catch (error) {
          console.error("Error fetching data:", error);
      }
  };

  onMount(async () => {
      await fetchData();

      fetchCountryData();

      await fetchCountries();

      fetchCurrency();
      fetchTimeZone();
      showLogoButton();
      const inputElement = document.querySelector("#phone-input");

      iti = intlTelInput(inputElement, {
          utilsScript:
              "https://cdn.jsdelivr.net/npm/intl-tel-input@16.0.3/build/js/utils.js",
          nationalMode: true,
          initialCountry: preferredCountryCode,
          preferredCountries: selectedLabels,
      });
      iti.setNumber(phone);
  });

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

  async function fetchCountries() {
      try {
          const response = await fetch(
              "https://api.recruitly.io/api/lookup/countries?apiKey=TEST45684CB2A93F41FC40869DC739BD4D126D77"
          );
          const responseData = await response.json();

          if (Array.isArray(responseData.data)) {
              countries = responseData.data.map((country) => ({
                  name: country.name,
                  value: country.code.toLowerCase(),
              }));
              selectedCodes = countries
                  .filter((country) => selectedLabels.includes(country.value))
                  .map((selectedCountry) => selectedCountry.value);
          } else {
              console.error("Invalid API response format:", responseData);
          }
      } catch (error) {
          console.error("Error fetching data:", error);
      }
  }

  async function fetchTimeZone() {
      try {
          const response = await fetch(
              "https://api.recruitly.io/api/lookup/timezone?apiKey=TEST45684CB2A93F41FC40869DC739BD4D126D77"
          );

          const responseData = await response.json();

          if (Array.isArray(responseData.data)) {
              reactiveTimezoneOptions = responseData.data.map((timeZone) => ({
                  value: timeZone.code,
                  name: timeZone.name,
              }));
          } else {
              console.error("Invalid currency data format:", responseData);
          }
      } catch (error) {
          console.error("Error fetching currency options:", error);
      }
  }

  async function fetchCurrency() {
      try {
          const response = await fetch(
              "https://api.recruitly.io/api/lookup/currency?apiKey=TEST45684CB2A93F41FC40869DC739BD4D126D77"
          );
          const responseData = await response.json();

          if (Array.isArray(responseData.data)) {
              // Map the response data to the format required for options
              currencyOptions = responseData.data.map((currency) => ({
                  value: currency.code,
                  name: currency.name,
              }));
          } else {
              console.error("Invalid currency data format:", responseData);
          }
      } catch (error) {
          console.error("Error fetching currency options:", error);
      }
  }
  function handleSelection(event) {
      if (event && event.detail && event.detail.value) {
          selectedCodes = event.detail.value.map((selectedCountry) =>
              selectedCountry.value.toLowerCase()
          );
      }
  }

  function validateForm() {
      errors = {};

      if (!name || name.trim() === "") {
          errors.name = "Company Name is required";
      }

      if (!cityName || cityName.trim() === "") {
          errors.cityName = "City Name is required";
      }
      if (!selectedCountry || selectedCountry.trim() === "") {
          errors.selectedCountry = "Country Name is required";
      }

      const phoneRegex = /^[0-9+]+$/;
      if (phone.trim() === "" || !phone) {
          errors.phone = "phone is required";
      } else if (!phoneRegex.test(phone)) {
          errors.phone = "phone Number is in incorrect format";
      }

      // Validate Website

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

  export async function updateData() {
      phone = iti.getNumber();

      const dataToUpdate = {
          logo,
          id,
          name,

          address: {
              addressLine: addressLine,
              cityName: cityName,
              regionName: regionName,
              postCode: postCode,
              countryCode: selectedCountry,
              latitude: latitude,
              longitude: longitude,
          },
          phone,
          website,
          currencyCode: {
              code: currencyCode,
          },
          preferredDateformat,
          timeZone,
          preferredCountries: selectedLabels,
          preferredCountryCode: preferredCountryCode1.toLowerCase(),
      };
      console.log(JSON.stringify(dataToUpdate));

      fetch(
          "https://api.recruitly.io/api/business/profile/save?apiKey=TEST45684CB2A93F41FC40869DC739BD4D126D77",
          {
              method: "POST",
              headers: {
                  "Content-Type": "application/json",
                  "Access-Control-Origin": "*",
              },
              body: JSON.stringify(dataToUpdate),
          }
      ).then((response) => {
          console.log(response, "resp");
          if (!response.ok) {
              throw new Error("Network response was not ok");
          }
          return response.json();
      });
  }

  const handleLogoChange = (event) => {
      file = event.target.files[0];
      if (file) {
          const reader = new FileReader();

          reader.onload = () => {
              logo = reader.result;

              showLogoButton();
          };

          reader.readAsDataURL(file);
      }
  };
  const showLogoButton = () => {
      const inputElement = document.getElementById("logoInput");
      const logoImage = document.getElementById("logoImage");

      if (file || logo) {
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
      formData.append("type", "TENANT");

      fetch(
          "https://api.recruitly.io/api/image/upload?apiKey=" +
              appData.service.apiKey,
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
</script>

<main class="container">
  <div class="form-container">
      <form on:submit|preventDefault>
          <div class="card card-fluid shadow-none border">
              <h6 class="card-header">Company Profile</h6>
          </div>
          <div style="padding-left:25px">
              <div
                  class="mb-3 business-logo-container"
                  style="padding-top: 20px"
                  on:change={uploadImage}
              >
                  <label for="CompanyLogo" class="mb-2">Company Logo</label>

                  <input
                      type="file"
                      accept="image/*"
                      id="logoInput"
                      class="file-input"
                      on:change={handleLogoChange}
                  />

                  <!-- svelte-ignore a11y-click-events-have-key-events -->
                  <!-- Update the image element in your Svelte code -->
                  <img
                      alt
                      src={logo}
                      id="logoImage"
                      on:click={() => {
                          document.getElementById("logoInput").click();
                      }}
                  />
              </div>
              <div class="mb-6">
                  <Label for="name" class="mb-2" style="padding-top: 20px"
                      >Company Name<span class="required">*</span></Label
                  >
                  <Input
                      type="text"
                      id="name"
                      bind:value={name}
                      placeholder="Company Name"
                  />
                  {#if showValidation && errors.name}
                      <p class="error">{errors.name}</p>
                  {/if}
              </div>

              <div class="mb-3">
                  <Label for="cityName" class="mb-2"
                      >City Name<span class="required">*</span></Label
                  >
                  <Input
                      type="text"
                      id="cityName"
                      bind:value={cityName}
                      placeholder="CityName"
                  />
                  {#if showValidation && errors.cityName}
                      <p class="error">{errors.cityName}</p>
                  {/if}
              </div>
              <div class="mb-3">
                  <Label for="country" class="mb-2"
                      >Country<span class="required">*</span></Label
                  >
                  <Select
                      class="mt-2"
                      items={countryOptions}
                      bind:value={selectedCountry}
                  />
                  {#if showValidation && errors.selectedCountry}
                      <p class="error">{errors.selectedCountry}</p>
                  {/if}
              </div>
              <div class="mb-3">
                  <Label for="postCode" class="mb-2">Post Code</Label>
                  <Input
                      type="text"
                      id="postCode"
                      bind:value={postCode}
                      placeholder="Post Code"
                  />
              </div>

              <div class="mb-3">
                  <Label for="phone" class="mb-2"
                      >Phone<span class="required">*</span></Label
                  >
                  <Input
                      id="phone-input"
                      type="tel"
                      bind:value={phone}
                      class="form-select {!isValid &&
                          'invalid'} block w-full py-2.5 pl-3 pr-10 text-base border border-gray-300 rounded-lg bg-transparent focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm dark:text-white dark:focus:ring-gray-700 dark:focus:border-gray-600"
                      style="width:775px ;"
                  />
                  {#if showValidation && errors.phone}
                      <p class="error">{errors.phone}</p>
                  {/if}
              </div>
              <div class="mb-3">
                  <Label for="website" class="mb-2">Website</Label>
                  <Input
                      type="text"
                      id="website"
                      bind:value={website}
                      placeholder="Website"
                  />
              </div>
              <div class="mb-3">
                  <Label for="currency" class="mb-2">Currency</Label>

                  <Select
                      class="mt-2"
                      items={currencyOptions}
                      bind:value={currencyCode}
                  />
              </div>

              <div class="mb-3">
                  <Label for="preferredDateformat" class="mb-2"
                      >Preferred Date Format</Label
                  >
                  <Select
                      class="block w-full rounded-lg bg-white border border-gray-300 text-gray-700 focus:outline-none focus:border-indigo-500"
                      bind:value={preferredDateformat}
                      placeholder="choose dateformat"
                  >
                      <option value="">Preferred Date Format</option>
                      <option value="default"
                          >Pretty(eg.1 Day Ago/2 Week Ago etc.,)
                      </option>
                      <option value="dateOnly"
                          >Date Only(eg.12/31/2023)</option
                      >
                      <option value="dateWithTime"
                          >Date with Time(eg.12/31/2023 15:00:00)
                      </option>
                      <option value="dateWithYear"
                          >Date with Year(eg.12/31/2023 03:00 PM)
                      </option>
                  </Select>
              </div>

              <div class="mb-3">
                  <Label for="timeZone" class="mb-2">Time Zone</Label>
                  <Select
                      class="mt-2"
                      items={reactiveTimezoneOptions}
                      bind:value={timeZone}
                  />
              </div>

              <div class="mb-3">
                  <Label for="country" class="mb-2"
                      >Default Country To Display</Label
                  >
                  <Select
                      class="mt-2"
                      items={countryOptions}
                      bind:value={preferredCountryCode1}
                  />
              </div>

              <div class="mb-3">
                  <Label class="mb-2"
                      >Preferred Countries To Display on Top</Label
                  >

                  {#if countries.length > 0}
                      <MultiSelect
                          items={countries}
                          bind:value={selectedLabels}
                          on:change={handleSelection}
                      />
                  {/if}
              </div>

              <Button
                  class="btn btn-primary"
                  type="submit"
                  on:click={handleSubmit}
                  style="background-color: #0545a6 !important;margin-top:15px"
              >
                  Update Profile
              </Button>
          </div>
      </form>
  </div>
</main>

<style>
  .card-header {
      font-weight: 600;
  }
  .card-header {
      width: 800px;
      margin-left: 0px;
      padding-top: 1rem;
      padding-bottom: 1rem;

      margin-bottom: 0;
      background-color: transparent;
      border-bottom: 2px solid rgba(20, 20, 31, 0.12);
  }
  h6 {
      font-size: 1rem;
      padding-left: 15px;
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
      max-width: 600px;
      max-height: 100px;
  }
  .required {
      color: red;
      margin-left: 2px;
  }
  .container {
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100vh;
  }

  .form-container {
      width: 800px;
      /* padding-left: 50px;
  padding-right:50px; */
      /* padding: 50px; */
      border: 1px solid #f9f9f9;
      border-radius: 5px;
      background-color: #f9f9f9;
      margin-top: 300px;
  }
</style>
