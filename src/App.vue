<script setup>
import Layout from "./layout/Layout.vue";
import Welcome from "./pages/Welcome.vue";
import Dashboard from "./pages/Dashboard.vue";
import Workout from "./pages/Workout.vue";
import { ref, computed, onMounted } from "vue";
import { workoutProgram } from "./utils";

const defaultData = {};

for (let index in workoutProgram) {
  const woukoutData = workoutProgram[index];
  defaultData[index] = {};

  for (let e of woukoutData.workout) {
    defaultData[index][e.name] = "";
  }
}
const selectedDisplay = ref(1);
const data = ref(defaultData);
const selectedWorkout = ref(-1);

const isWorkoutComplete = computed(() => {
  const currWorkout = data.value?.[selectedWorkout.value];
  if (!currWorkout) return false;

  const isCompleteCheck = Object.values(currWorkout).every(
    (exercise) => !!exercise
  );

  console.log("isCompleteCheck", isCompleteCheck);
  return isCompleteCheck;
});

const firstIncompleteWorkoutIndex = computed(() => {
  const allWorkouts = data.value;
  if (!allWorkouts) return -1;

  for (const [index, workout] of Object.entries(allWorkouts)) {
    const isComplete = Object.values(workout).every((exercise) => !!exercise);
    if (!isComplete) {
      return parseInt(index);
    }
  }

  return -1;
});

function handleChangeDisplay(index) {
  selectedDisplay.value = index;
}

function handleSelectWorkout(index) {
  selectedDisplay.value = 3;
  selectedWorkout.value = index;
}

function handleSaveWorkout() {
  localStorage.setItem("workouts", JSON.stringify(data.value));
  selectedDisplay.value = 2;
  selectedWorkout.value = -1;
}

function handleResetPlan() {
  selectedDisplay.value = 2;
  selectedWorkout.value = -1;
  data.value = defaultData;
  localStorage.removeItem("workouts");
}

onMounted(() => {
  if (!localStorage) return;

  if (localStorage.getItem("workouts")) {
    const savedData = JSON.parse(localStorage.getItem("workouts"));
    data.value = savedData;
    selectedDisplay.value = 2;
  }
});
</script>

<template>
  <Layout>
    <Welcome
      :handleChangeDisplay="handleChangeDisplay"
      v-if="selectedDisplay === 1"
    />
    <Dashboard
      :handleResetPlan="handleResetPlan"
      :firstIncompleteWorkoutIndex="firstIncompleteWorkoutIndex"
      :handleSelectWorkout="handleSelectWorkout"
      v-if="selectedDisplay === 2"
    />
    <Workout
      :handleSaveWorkout="handleSaveWorkout"
      :isWorkoutComplete="isWorkoutComplete"
      :data="data"
      :selectedWorkout="selectedWorkout"
      v-if="workoutProgram?.[selectedWorkout]"
    />
  </Layout>
</template>

<style scoped></style>
