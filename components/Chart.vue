<script setup>
import { watch, ref, computed, onMounted } from "vue";

let props = defineProps(["currentCategory", "data"]);
let data = props.data || [];
let currentCategory = props.currentCategory || "today";

let categories = ref({
  today: [],
  week: [
    "Sunday",
    "Monday",
    "Tuesday",
    "Wednesday",
    "Thursday",
    "Friday",
    "Saturday",
  ],
  month: [], // Will be generated dynamically
  year: Array.from({ length: 365 }, (_, i) => (i + 1).toString()),
});

// Function to generate 24 hours for today
function generateTodayCategories() {
  categories.value.today = Array.from(
    { length: 24 },
    (_, i) => `${i.toString().padStart(2, "0")}:00`
  );
}

// Function to generate the current month days
function generateMonthCategories() {
  let currentDate = new Date();
  let currentMonth = currentDate.getMonth() + 1;
  let currentYear = currentDate.getFullYear();

  let monthDates = [];
  let daysInMonth = new Date(currentYear, currentMonth, 0).getDate();
  for (let i = 1; i <= daysInMonth; i++) {
    let dayString = ("0" + i).slice(-2); // Format day with two digits (e.g., 01, 02, ...)
    let monthString = ("0" + currentMonth).slice(-2); // Format month
    monthDates.push(`${monthString}/${dayString}`);
  }
  categories.value.month = monthDates;
}

// Watch for category changes
watch(
  () => props.currentCategory,
  (newCategory) => {
    // Update chart categories dynamically when switching tabs
    if (newCategory === "today") {
      generateTodayCategories(); // Regenerate 24-hour categories when switching to "today"
    } else if (newCategory === "week") {
      categories.value.week = [
        "Sunday",
        "Monday",
        "Tuesday",
        "Wednesday",
        "Thursday",
        "Friday",
        "Saturday",
      ];
    } else if (newCategory === "month") {
      generateMonthCategories(); // Regenerate categories when switching to the "month" tab
    } else if (newCategory === "year") {
      categories.value.year = Array.from({ length: 365 }, (_, i) =>
        (i + 1).toString()
      );
    }
  },
  { immediate: true }
);

// Initialize default categories on mount
onMounted(() => {
  generateTodayCategories(); // Generate categories for "today" on mount
});

let options = computed(() => ({
  chart: {
    type: "line",
    animation: {
      enabled: false,
    },
  },
  title: {
    text: "",
  },
  xAxis: {
    gridLineColor: "transparent", // Remove gridlines
    categories: categories.value[currentCategory], // Set xAxis categories dynamically based on the current category
  },
  yAxis: {
    gridLineColor: "transparent",
    title: {
      text: "", // No title for yAxis
    },
  },
  legend: {
    enabled: false, // Disable legend
  },
  plotOptions: {
    line: {
      marker: {
        enabled: false, // No markers on the line
      },
      dataLabels: {
        enabled: false,
      },
      enableMouseTracking: false,
    },
  },
  series: [
    {
      name: "line",
      lineWidth: "4px",
      color: {
        linearGradient: { y1: 0, y2: 0, y3: 0, y4: 0 },
        stops: [
          [0, "rgba(252,176,69,1)"],
          [0.33, "rgba(253,29,29,1)"],
          [0.66, "rgba(131,58,180,1)"],
          [1, "rgba(29,217,93,1)"],
        ],
      },
      data: props.data,
    },
  ],
}));
</script>

<template>
  <div class="border rounded-lg p-4">
    <highchart :options="options" />
  </div>
</template>
