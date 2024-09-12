<script>
  import { onMount } from "svelte";

  let scenarios = [];
  let descriptions = []; // Array to hold love language descriptions
  let currentScenarios = [];
  let selectedAnswers = [];
  let usedScenarioIds = new Set(); // Set to track used scenario ids
  let questionIndex = 0;
  let quizComplete = false;
  let loveLanguageCounts = {};
  let totalSelections = 0;
  let totalQuestions = 0;
  let topLoveLanguage = null; // Variable to store the top love language

  // Shuffle function to randomize the array
  function shuffleArray(array) {
    for (let i = array.length - 1; i > 0; i--) {
      const j = Math.floor(Math.random() * (i + 1));
      [array[i], array[j]] = [array[j], array[i]];
    }
  }

  // Fetch the scenarios and descriptions from the JSON files
  onMount(async () => {
    const scenarioResponse = await fetch("/data/scenarios.json");
    scenarios = await scenarioResponse.json();
    
    const descriptionResponse = await fetch("/data/descriptions.json");
    descriptions = await descriptionResponse.json();

    totalQuestions = Math.floor(scenarios.length / 2); // Total questions is half of the scenario count since we're pairing them
    shuffleArray(scenarios); // Shuffle scenarios to avoid repetition
    loadNextQuestion();
  });

  // Load the next two scenarios for the question, ensuring no repeats and different love languages
  function loadNextQuestion() {
    if (scenarios.length > 0 && questionIndex < totalQuestions) {
      if (questionIndex === totalQuestions - 1) {
        // All questions are done
        quizComplete = true;
        calculateResults();
      } else {
        let scenario1, scenario2;
        do {
          scenario1 = scenarios.find((s) => !usedScenarioIds.has(s.id));
        } while (!scenario1);

        do {
          scenario2 = scenarios.find(
            (s) => s.id !== scenario1.id && s.Language !== scenario1.Language && !usedScenarioIds.has(s.id),
          );
        } while (!scenario2);

        // Add these scenarios' ids to the used set
        usedScenarioIds.add(scenario1.id);
        usedScenarioIds.add(scenario2.id);

        currentScenarios = [scenario1, scenario2];
        questionIndex++;
      }
    }
  }

  // Function to handle answer selection and load the next question
  function selectScenario(selected) {
    selectedAnswers.push(selected);
    totalSelections++;
    loadNextQuestion();
  }

  // Calculate the percentage of each love language selected
  function calculateResults() {
    loveLanguageCounts = {};

    selectedAnswers.forEach((answer) => {
      if (loveLanguageCounts[answer.Language]) {
        loveLanguageCounts[answer.Language]++;
      } else {
        loveLanguageCounts[answer.Language] = 1;
      }
    });

    // Convert counts to percentages
    let maxPercentage = 0;
    for (let language in loveLanguageCounts) {
      loveLanguageCounts[language] = Math.round(
        (loveLanguageCounts[language] / totalSelections) * 100,
      );

      // Find the top love language
      if (loveLanguageCounts[language] > maxPercentage) {
        maxPercentage = loveLanguageCounts[language];
        topLoveLanguage = language;
      }
    }
  }

  // Get the description of the love language from the descriptions array
  function getLoveLanguageDescription(language) {
    return descriptions.find((desc) => desc.language === language);
  }
</script>

<main class="container mt-5">
  {#if !quizComplete && currentScenarios.length === 2}
    <h1 class="text-center mb-4">I feel most loved when:</h1>
    <!-- Display the question index like "Question {X} of {Y}" -->
    <p class="text-center">Question {questionIndex} of {totalQuestions}</p>
    <div class="quiz-container row justify-content-center">
      <div class="col-12 col-md-6 mb-3 d-flex justify-content-center">
        <button
          class="btn btn-primary btn-lg w-100 equal-height"
          on:click={() => selectScenario(currentScenarios[0])}
        >
          {currentScenarios[0].Scenario}
        </button>
      </div>
      <div class="col-12 col-md-6 d-flex justify-content-center">
        <button
          class="btn btn-secondary btn-lg w-100 equal-height"
          on:click={() => selectScenario(currentScenarios[1])}
        >
          {currentScenarios[1].Scenario}
        </button>
      </div>
    </div>
  {:else if quizComplete}
    <p class="text-center">You've completed the quiz! Here are your results:</p>
    <ul class="list-group list-group-flush">
      {#each Object.keys(loveLanguageCounts) as language}
        <li class="list-group-item">
          <strong>{language}</strong>: {loveLanguageCounts[language]}%
        </li>
      {/each}
    </ul>

    <!-- Section to display the top love language and its description -->
    {#if topLoveLanguage}
      <div class="top-love-language mt-4 p-3">
        <h2 class="text-center">Your top love language is: <strong>{topLoveLanguage}</strong></h2>
        {#if getLoveLanguageDescription(topLoveLanguage)}
          <p><strong>Description:</strong> {getLoveLanguageDescription(topLoveLanguage).description}</p>
          <p><strong>How to Communicate:</strong> {getLoveLanguageDescription(topLoveLanguage).howToCommunicate}</p>
          <p><strong>Actions to Take:</strong> {getLoveLanguageDescription(topLoveLanguage).actionsToTake}</p>
          <p><strong>Things to Avoid:</strong> {getLoveLanguageDescription(topLoveLanguage).thingsToAvoid}</p>
        {/if}
      </div>
    {/if}
  {:else}
    <p class="text-center">Loading...</p>
  {/if}
  <p class="footer-text">byLfz</p>
</main>

<style>
  .quiz-container {
    margin-top: 20px;
  }
  .equal-height {
    height: 120px;
  }
  button {
    padding: 15px;
    font-size: 18px;
    cursor: pointer;
  }
  .footer-text {
    position: fixed;
    bottom: 10px;
    right: 10px;
    font-size: 14px;
    color: #555;
  }
  .top-love-language {
    background-color: #f9f9f9;
    border: 1px solid #ddd;
    border-radius: 5px;
    padding: 20px;
  }
</style>
