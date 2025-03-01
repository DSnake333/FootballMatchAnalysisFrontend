<template>
  <div :class="{'dark-mode': isDarkMode}">
    <!-- Header (always visible) -->
    <header class="header">
      <h1>
        <i class="fas fa-futbol"></i> Soccer Data Analysis
      </h1>
      <button @click="toggleDarkMode" class="dark-mode-toggle">
        <i :class="isDarkMode ? 'fas fa-sun' : 'fas fa-moon'"></i>
        {{ isDarkMode ? 'Light Mode' : 'Dark Mode' }}
      </button>
    </header>

    <!-- URL Input Form (always visible) -->
    <div class="url-input-container">
      <form @submit.prevent="submitUrl" class="url-form">
        <input
          v-model="matchUrl"
          type="url"
          placeholder="Enter fbref.com match URL"
          required
          class="url-input"
        />
        <button type="submit" class="submit-button" :disabled="isLoading">
          <i class="fas fa-search"></i> {{ isLoading ? 'Analyzing...' : 'Analyze Match' }}
        </button>
      </form>
    </div>

    <!-- Loading Spinner -->
    <div v-if="isLoading" class="loading-spinner">
      <i class="fas fa-spinner fa-spin"></i> Loading...
    </div>

    <!-- Message when no data is loaded -->
    <div v-else-if="!hasData" class="no-data-message">
      <p>Enter a match URL above to start the analysis</p>
    </div>

    <!-- Main Content (only visible when data is loaded) -->
    <main v-if="hasData" class="main-content">
      <!-- Keeper Stats Analysis Section -->
      <section id="keeper-analysis" class="section">
        <h2>
          <i class="fas fa-chess-king"></i> Keeper Stats Analysis
        </h2>
        <p>
          This section provides insights into goalkeeper performance, including save efficiency, cross control, and workload handling.
        </p>
        <transition name="fade" mode="out-in">
          <div v-if="saveEfficiencyData" class="metric-card">
            <h3><i class="fas fa-chart-line"></i> Save Efficiency</h3>
            <KeeperMetrics :plotlyData="saveEfficiencyData" />
            <p>Explanation: This chart shows how efficient goalkeepers are at saving shots based on their expected goals faced (PSxG).</p>
          </div>
        </transition>
        <transition name="fade" mode="out-in">
          <div v-if="crossControlSuccessData" class="metric-card">
            <h3><i class="fas fa-crosshairs"></i> Cross Control Success</h3>
            <KeeperMetrics :plotlyData="crossControlSuccessData" />
            <p>Explanation: This chart highlights the ability of goalkeepers to control crosses into their penalty area.</p>
          </div>
        </transition>
        <transition name="fade" mode="out-in">
          <div v-if="passEffectivenessData" class="metric-card">
            <h3><i class="fas fa-paper-plane"></i> Pass Effectiveness</h3>
            <KeeperMetrics :plotlyData="passEffectivenessData" />
            <p>Explanation: This chart evaluates the effectiveness of goalkeepers' passing under pressure.</p>
          </div>
        </transition>
        <transition name="fade" mode="out-in">
          <div v-if="defensiveActionsData" class="metric-card">
            <h3><i class="fas fa-shield-alt"></i> Defensive Actions</h3>
            <KeeperMetrics :plotlyData="defensiveActionsData" />
            <p>Explanation: This chart shows the number of defensive actions (e.g., clearances, blocks) by goalkeepers.</p>
          </div>
        </transition>
        <transition name="fade" mode="out-in">
          <div v-if="defensiveWorkloadData" class="metric-card">
            <h3><i class="fas fa-tachometer-alt"></i> Defensive Workload</h3>
            <KeeperMetrics :plotlyData="defensiveWorkloadData" />
            <p>Explanation: This chart highlights the workload of goalkeepers in terms of defensive actions per game.</p>
          </div>
        </transition>
      </section>

      <!-- Possession Analysis Section -->
      <section id="possession-analysis" class="section">
        <h2>
          <i class="fas fa-running"></i> Possession Analysis
        </h2>
        <p>
          This section analyzes possession metrics, including carries, progressive pass receptions, and dispossessions.
        </p>
        <transition name="fade" mode="out-in">
          <div v-if="carryTakeOnData" class="metric-card">
            <h3><i class="fas fa-walking"></i> Carry & Take On</h3>
            <KeeperMetrics :plotlyData="carryTakeOnData" />
            <p>Explanation: This chart displays players' ability to carry the ball and successfully take on opponents.</p>
          </div>
        </transition>
        <transition name="fade" mode="out-in">
          <div v-if="progressivePassReceptionData" class="metric-card">
            <h3><i class="fas fa-exchange-alt"></i> Progressive Pass Reception</h3>
            <KeeperMetrics :plotlyData="progressivePassReceptionData" />
            <p>Explanation: This chart shows how effectively players receive progressive passes.</p>
          </div>
        </transition>
        <transition name="fade" mode="out-in">
          <div v-if="touchDistributionData" class="metric-card">
            <h3><i class="fas fa-hand-paper"></i> Touch Distribution</h3>
            <KeeperMetrics :plotlyData="touchDistributionData" />
            <p>Explanation: This chart shows the distribution of touches across different areas of the field.</p>
          </div>
        </transition>
        <transition name="fade" mode="out-in">
          <div v-if="miscontrolsDispossessionsData" class="metric-card">
            <h3><i class="fas fa-exclamation-triangle"></i> Miscontrols & Dispossessions</h3>
            <KeeperMetrics :plotlyData="miscontrolsDispossessionsData" />
            <p>Explanation: This chart highlights the frequency of miscontrols and dispossessions by players.</p>
          </div>
        </transition>
      </section>

      <!-- Defense Stats Analysis Section -->
      <section id="defense-analysis" class="section">
        <h2>
          <i class="fas fa-shield-alt"></i> Defense Stats Analysis
        </h2>
        <p>
          This section evaluates defensive contributions, such as tackles, interceptions, and clearances.
        </p>
        <transition name="fade" mode="out-in">
          <div v-if="tacklingEfficiencyData" class="metric-card">
            <h3><i class="fas fa-hand-rock"></i> Tackling Efficiency</h3>
            <KeeperMetrics :plotlyData="tacklingEfficiencyData" />
            <p>Explanation: This chart displays players' ability to carry the ball and successfully take on opponents.</p>
          </div>
        </transition>
        <transition name="fade" mode="out-in">
          <div v-if="topDefendersData" class="metric-card">
            <h3><i class="fas fa-star"></i> Top Defenders</h3>
            <KeeperMetrics :plotlyData="topDefendersData" />
            <p>Explanation: This chart highlights the top defenders based on key defensive metrics.</p>
          </div>
        </transition>
        <transition name="fade" mode="out-in">
          <div v-if="dribblerStopRateData" class="metric-card">
            <h3><i class="fas fa-ban"></i> Dribbler Stop Rate</h3>
            <KeeperMetrics :plotlyData="dribblerStopRateData" />
            <p>Explanation: This chart shows how often defenders successfully stop dribblers.</p>
          </div>
        </transition>
        <transition name="fade" mode="out-in">
          <div v-if="defensivePressureData" class="metric-card">
            <h3><i class="fas fa-fire"></i> Defensive Pressure</h3>
            <KeeperMetrics :plotlyData="defensivePressureData" />
            <p>Explanation: This chart evaluates the amount of defensive pressure applied by players.</p>
          </div>
        </transition>
        <transition name="fade" mode="out-in">
          <div v-if="playersDefensiveImpactData" class="metric-card">
            <h3><i class="fas fa-fist-raised"></i> Defensive Impact</h3>
            <KeeperMetrics :plotlyData="playersDefensiveImpactData" />
            <p>Explanation: This chart measures the overall defensive impact of players.</p>
          </div>
        </transition>
      </section>

      <!-- Shots Stats Analysis Section -->
      <section id="shots-analysis" class="section">
        <h2>
          <i class="fas fa-bullseye"></i> Shots Stats Analysis
        </h2>
        <p>
          This section evaluates shooting metrics, including shot outcomes, xG heatmaps, and shooting efficiency.
        </p>
        <transition name="fade" mode="out-in">
          <div v-if="shotOutcomesData" class="metric-card">
            <h3><i class="fas fa-bullseye"></i> Shot Outcomes</h3>
            <KeeperMetrics :plotlyData="shotOutcomesData" />
            <p>Explanation: This chart displays the outcomes of shots taken by players.</p>
          </div>
        </transition>
        <transition name="fade" mode="out-in">
          <div v-if="xgHeatMapData" class="metric-card">
            <h3><i class="fas fa-map"></i> xG Heat Map</h3>
            <KeeperMetrics :plotlyData="xgHeatMapData" />
            <p>Explanation: This chart visualizes the expected goals (xG) heatmap for shots taken.</p>
          </div>
        </transition>
        <transition name="fade" mode="out-in">
          <div v-if="shotDistanceData" class="metric-card">
            <h3><i class="fas fa-ruler"></i> Shot Distance</h3>
            <KeeperMetrics :plotlyData="shotDistanceData" />
            <p>Explanation: This chart shows the distribution of shot distances.</p>
          </div>
        </transition>
        <transition name="fade" mode="out-in">
          <div v-if="shootingEfficiencyData" class="metric-card">
            <h3><i class="fas fa-percentage"></i> Shooting Efficiency</h3>
            <KeeperMetrics :plotlyData="shootingEfficiencyData" />
            <p>Explanation: This chart evaluates the shooting efficiency of players.</p>
          </div>
        </transition>
        <transition name="fade" mode="out-in">
          <div v-if="topPlayersScaData" class="metric-card">
            <h3><i class="fas fa-trophy"></i> Top Players SCA</h3>
            <KeeperMetrics :plotlyData="topPlayersScaData" />
            <p>Explanation: This chart highlights the top players based on shot-creating actions (SCA).</p>
          </div>
        </transition>
      </section>

      <!-- Passing Stats Analysis Section -->
      <section id="passing-analysis" class="section">
        <h2>
          <i class="fas fa-passport"></i> Passing Stats Analysis
        </h2>
        <p>
          This section evaluates passing metrics, including pass types, progressive passes, and key passes.
        </p>
        <transition name="fade" mode="out-in">
          <div v-if="teamPassingData" class="metric-card">
            <h3><i class="fas fa-network-wired"></i> Team Passing</h3>
            <KeeperMetrics :plotlyData="teamPassingData" />
            <p>Explanation: This chart displays the overall passing metrics for the team.</p>
          </div>
        </transition>
        <transition name="fade" mode="out-in">
          <div v-if="passTypeData" class="metric-card">
            <h3><i class="fas fa-project-diagram"></i> Pass Types</h3>
            <KeeperMetrics :plotlyData="passTypeData" />
            <p>Explanation: This chart breaks down the types of passes made by players.</p>
          </div>
        </transition>
        <transition name="fade" mode="out-in">
          <div v-if="topPassingData" class="metric-card">
            <h3><i class="fas fa-star"></i> Top Passers</h3>
            <KeeperMetrics :plotlyData="topPassingData" />
            <p>Explanation: This chart highlights the top passers based on accuracy and volume.</p>
          </div>
        </transition>
        <transition name="fade" mode="out-in">
          <div v-if="progressivePassersData" class="metric-card">
            <h3><i class="fas fa-arrow-up"></i> Progressive Passers</h3>
            <KeeperMetrics :plotlyData="progressivePassersData" />
            <p>Explanation: This chart evaluates players who make the most progressive passes.</p>
          </div>
        </transition>
        <transition name="fade" mode="out-in">
          <div v-if="keyPassersData" class="metric-card">
            <h3><i class="fas fa-key"></i> Key Passers</h3>
            <KeeperMetrics :plotlyData="keyPassersData" />
            <p>Explanation: This chart highlights players who create the most key passes.</p>
          </div>
        </transition>
        <transition name="fade" mode="out-in">
          <div v-if="penetrativePassersData" class="metric-card">
            <h3><i class="fas fa-bullseye"></i> Penetrative Passers</h3>
            <KeeperMetrics :plotlyData="penetrativePassersData" />
            <p>Explanation: This chart evaluates players who make the most penetrative passes.</p>
          </div>
        </transition>
        <transition name="fade" mode="out-in">
          <div v-if="longPassersData" class="metric-card">
            <h3><i class="fas fa-long-arrow-alt-right"></i> Long Passers</h3>
            <KeeperMetrics :plotlyData="longPassersData" />
            <p>Explanation: This chart highlights players who excel at long passes.</p>
          </div>
        </transition>
        <transition name="fade" mode="out-in">
          <div v-if="advancedPassData" class="metric-card">
            <h3><i class="fas fa-chart-line"></i> Advanced Pass Metrics</h3>
            <KeeperMetrics :plotlyData="advancedPassData" />
            <p>Explanation: This chart evaluates advanced passing metrics.</p>
          </div>
        </transition>
        <transition name="fade" mode="out-in">
          <div v-if="cornerKicksData" class="metric-card">
            <h3><i class="fas fa-flag"></i> Corner Kicks</h3>
            <KeeperMetrics :plotlyData="cornerKicksData" />
            <p>Explanation: This chart analyzes corner kick performance.</p>
          </div>
        </transition>
      </section>

      <!-- Miscellaneous Stats Analysis Section -->
      <section id="misc-analysis" class="section">
        <h2>
          <i class="fas fa-chart-pie"></i> Miscellaneous Stats Analysis
        </h2>
        <p>
          This section evaluates miscellaneous metrics, such as fouls, aerial duels, and defensive impact.
        </p>
        <transition name="fade" mode="out-in">
          <div v-if="tackleInterceptionEfficiencyData" class="metric-card">
            <h3><i class="fas fa-hand-rock"></i> Tackles & Interceptions</h3>
            <KeeperMetrics :plotlyData="tackleInterceptionEfficiencyData" />
            <p>Explanation: This chart evaluates the efficiency of tackles and interceptions.</p>
          </div>
        </transition>
        <transition name="fade" mode="out-in">
          <div v-if="foulsEfficiencyData" class="metric-card">
            <h3><i class="fas fa-exclamation-circle"></i> Fouls Efficiency</h3>
            <KeeperMetrics :plotlyData="foulsEfficiencyData" />
            <p>Explanation: This chart evaluates the frequency and impact of fouls.</p>
          </div>
        </transition>
        <transition name="fade" mode="out-in">
          <div v-if="aerialEfficiencyData" class="metric-card">
            <h3><i class="fas fa-plane"></i> Aerial Efficiency</h3>
            <KeeperMetrics :plotlyData="aerialEfficiencyData" />
            <p>Explanation: This chart evaluates the success rate of aerial duels.</p>
          </div>
        </transition>
        <transition name="fade" mode="out-in">
          <div v-if="defensiveImpactData" class="metric-card">
            <h3><i class="fas fa-fist-raised"></i> Defensive Impact</h3>
            <KeeperMetrics :plotlyData="defensiveImpactData" />
            <p>Explanation: This chart measures the overall defensive impact of players.</p>
          </div>
        </transition>
        <transition name="fade" mode="out-in">
          <div v-if="playersDefensiveImpactData" class="metric-card">
            <h3><i class="fas fa-shield-alt"></i> Players Defensive Impact</h3>
            <KeeperMetrics :plotlyData="playersDefensiveImpactData" />
            <p>Explanation: This chart highlights the defensive impact of individual players.</p>
          </div>
        </transition>
      </section>
    </main>
    <!-- Back to Top Button -->
    <button
      v-if="showBackToTop"
      @click="scrollToTop"
      class="back-to-top"
      aria-label="Back to Top"
    >
      <i class="fas fa-arrow-up"></i>
    </button>
  </div>
</template>

<script>
import KeeperMetrics from "./components/KeeperMetrics.vue";

export default {
  name: 'App',
  components: {
    KeeperMetrics,
  },
  data() {
    return {
      isDarkMode: false,
      showBackToTop: false,
      matchUrl: "",
      isLoading: false,

      // Keeper Metrics Data
      saveEfficiencyData: null,
      crossControlSuccessData: null,
      passEffectivenessData: null,
      defensiveActionsData: null,
      defensiveWorkloadData: null,

      // Possession Metrics Data
      carryTakeOnData: null,
      progressivePassReceptionData: null,
      touchDistributionData: null,
      miscontrolsDispossessionsData: null,

      // Misc Metrics Data
      tackleInterceptionEfficiencyData: null,
      foulsEfficiencyData: null,
      aerialEfficiencyData: null,
      defensiveImpactData: null,

      // Defense Metrics Data
      tacklingEfficiencyData: null,
      topDefendersData: null,
      dribblerStopRateData: null,
      defensivePressureData: null,
      playersDefensiveImpactData: null,

      // Shot metrics data
      shotOutcomesData: null,
      xgHeatMapData: null,
      shotDistanceData: null,
      shootingEfficiencyData: null,
      topPlayersScaData: null,

      // Passing metrics data
      teamPassingData: null,
      passTypeData: null,
      topPassingData: null,
      progressivePassersData: null,
      keyPassersData: null,
      penetrativePassersData: null,
      longPassersData: null,
      advancedPassData: null,
      cornerKicksData: null,
    };
  },
  computed: {
    hasData() {
      return !!(
        this.saveEfficiencyData ||
        this.crossControlSuccessData ||
        this.passEffectivenessData ||
        this.defensiveActionsData ||
        this.defensiveWorkloadData ||
        this.carryTakeOnData ||
        this.progressivePassReceptionData ||
        this.touchDistributionData ||
        this.miscontrolsDispossessionsData ||
        this.tackleInterceptionEfficiencyData ||
        this.foulsEfficiencyData ||
        this.aerialEfficiencyData ||
        this.defensiveImpactData ||
        this.tacklingEfficiencyData ||
        this.topDefendersData ||
        this.dribblerStopRateData ||
        this.defensivePressureData ||
        this.playersDefensiveImpactData ||
        this.shotOutcomesData ||
        this.xgHeatMapData ||
        this.shotDistanceData ||
        this.shootingEfficiencyData ||
        this.topPlayersScaData ||
        this.teamPassingData ||
        this.passTypeData ||
        this.topPassingData ||
        this.progressivePassersData ||
        this.keyPassersData ||
        this.penetrativePassersData ||
        this.longPassersData ||
        this.advancedPassData ||
        this.cornerKicksData
      );
    }
  },
  methods: {
    toggleDarkMode() {
      this.isDarkMode = !this.isDarkMode;
    },
    handleScroll() {
      this.showBackToTop = window.scrollY > 300;
    },
    scrollToTop() {
      window.scrollTo({
        top: 0,
        behavior: "smooth",
      });
    },
    async submitUrl() {
      if (!this.matchUrl) {
        alert("Please enter a valid fbref.com match URL.");
        return;
      }

      this.isLoading = true;
      // Reset all data before new analysis
      this.resetData();

      try {
        const response = await fetch("http://localhost:8000/analyze-match", {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
            "Accept": "application/json",
          },
          body: JSON.stringify({ url: this.matchUrl }),
        });

        if (!response.ok) {
          throw new Error(await response.text());
        }

        const data = await response.json();
        console.log("Analysis completed:", data);
        await this.fetchMetrics();
      } catch (error) {
        console.error("Error:", error);
        alert(`Analysis failed: ${error.message}`);
      } finally {
        this.isLoading = false;
      }
    },
    resetData() {
      // Reset all data properties to null
      Object.keys(this.$data).forEach(key => {
        if (key !== 'isDarkMode' && key !== 'showBackToTop' && key !== 'matchUrl' && key !== 'isLoading') {
          this[key] = null;
        }
      });
    },
    async fetchMetrics() {
      try {
        const response = await fetch("http://localhost:8000/visualizations/metrics");
        if (!response.ok) {
          throw new Error('Failed to fetch metrics');
        }

        const data = await response.json();

        // Update all metrics with the new data
        this.saveEfficiencyData = data.save_efficiency;
        this.crossControlSuccessData = data.cross_control_success;
        this.passEffectivenessData = data.pass_effectiveness;
        this.defensiveActionsData = data.defensive_actions;
        this.defensiveWorkloadData = data.defensive_workload;
        this.carryTakeOnData = data.carry_take_on;
        this.progressivePassReceptionData = data.progressive_pass_reception;
        this.touchDistributionData = data.touch_distribution;
        this.miscontrolsDispossessionsData = data.miscontrols_dispossessions;
        this.tackleInterceptionEfficiencyData = data.tackle_interception;
        this.foulsEfficiencyData = data.fouls_efficiency;
        this.aerialEfficiencyData = data.aerial_efficiency;
        this.defensiveImpactData = data.defensive_impact;
        this.tacklingEfficiencyData = data.tackling_efficiency;
        this.topDefendersData = data.top_defenders;
        this.dribblerStopRateData = data.dribbler_stop_rate;
        this.defensivePressureData = data.defensive_pressure;
        this.playersDefensiveImpactData = data.players_defensive_impact;
        this.shotOutcomesData = data.shot_outcomes;
        this.xgHeatMapData = data.xg_heatmap;
        this.shotDistanceData = data.shot_distance;
        this.shootingEfficiencyData = data.shooting_efficiency;
        this.topPlayersScaData = data.top_players_sca;
        this.teamPassingData = data.team_passing_metrics;
        this.passTypeData = data.pass_type_distribution;
        this.topPassingData = data.top_passing_accuracy;
        this.progressivePassersData = data.progressive_passers;
        this.keyPassersData = data.key_passers;
        this.penetrativePassersData = data.penetrative_passers;
        this.longPassersData = data.long_pass_specialists;
        this.advancedPassData = data.advanced_pass_metrics;
        this.cornerKicksData = data.corner_kicks;
      } catch (error) {
        console.error("Error fetching metrics:", error);
        alert("Failed to fetch metrics. Please try again.");
      }
    },
  },
  mounted() {
    window.addEventListener("scroll", this.handleScroll);
  },
  beforeUnmount() {
    window.removeEventListener("scroll", this.handleScroll);
  },
};
</script>

<style>
/* Global Styles */
@import url('https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0-beta3/css/all.min.css');
@import url('https://fonts.googleapis.com/css2?family=Poppins:wght@300;400;500;600;700&display=swap');

body {
  font-family: 'Poppins', sans-serif;
  margin: 0;
  padding: 0;
  transition: background-color 0.3s, color 0.3s;
}

.dark-mode {
  background-color: #121212;
  color: #ffffff;
}

/* Header Styles */
.header {
  padding: 20px;
  text-align: center;
  background-color: #007bff;
  color: white;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

.dark-mode .header {
  background-color: #1a1a1a;
}

.header h1 {
  font-size: 2.5rem;
  margin: 0;
}

.header h1 i {
  margin-right: 10px;
}

/* URL Input Container Styles */
.url-input-container {
  max-width: 800px;
  margin: 2rem auto;
  padding: 0 1rem;
}

.url-form {
  display: flex;
  gap: 1rem;
}

.url-input {
  flex: 1;
  padding: 0.75rem;
  border: 2px solid #007bff;
  border-radius: 4px;
  font-size: 1rem;
  transition: border-color 0.3s;
}

.dark-mode .url-input {
  background-color: #2d2d2d;
  border-color: #bb86fc;
  color: white;
}

.submit-button {
  padding: 0.75rem 1.5rem;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 1rem;
  transition: background-color 0.3s, transform 0.2s;
}

.submit-button:hover {
  background-color: #0056b3;
  transform: scale(1.05);
}

.submit-button:disabled {
  background-color: #ccc;
  cursor: not-allowed;
  transform: none;
}

/* Loading Spinner Styles */
.loading-spinner {
  text-align: center;
  margin: 2rem 0;
  font-size: 1.5rem;
  color: #007bff;
}

.dark-mode .loading-spinner {
  color: #bb86fc;
}

/* No Data Message Styles */
.no-data-message {
  text-align: center;
  margin: 4rem 0;
  padding: 2rem;
  color: #666;
  font-size: 1.2rem;
  background-color: #f8f9fa;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

.dark-mode .no-data-message {
  background-color: #1e1e1e;
  color: #ccc;
}

/* Main Content Styles */
.main-content {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
}

/* Section Styles */
.section {
  margin-bottom: 40px;
  padding: 20px;
  background-color: white;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  transition: transform 0.3s;
}

.section:hover {
  transform: translateY(-5px);
}

.dark-mode .section {
  background-color: #1e1e1e;
}

/* Metric Card Styles */
.metric-card {
  margin-bottom: 20px;
  padding: 15px;
  background-color: #f8f9fa;
  border-radius: 8px;
  transition: transform 0.3s;
}

.metric-card:hover {
  transform: scale(1.02);
}

.dark-mode .metric-card {
  background-color: #2d2d2d;
}

/* Typography Styles */
h2 {
  font-size: 2rem;
  color: #007bff;
  margin-bottom: 1rem;
}

.dark-mode h2 {
  color: #bb86fc;
}

h3 {
  font-size: 1.5rem;
  color: #333;
  margin-bottom: 0.5rem;
}

.dark-mode h3 {
  color: #ffffff;
}

p {
  color: #666;
  line-height: 1.6;
}

.dark-mode p {
  color: #cccccc;
}

/* Back to Top Button Styles */
.back-to-top {
  position: fixed;
  bottom: 20px;
  right: 20px;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 50%;
  width: 50px;
  height: 50px;
  font-size: 1.2rem;
  cursor: pointer;
  transition: background-color 0.3s, transform 0.2s;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  z-index: 1000;
}

.back-to-top:hover {
  background-color: #0056b3;
  transform: scale(1.1);
}

.dark-mode .back-to-top {
  background-color: #bb86fc;
}

/* Animations */
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

/* Responsive Design */
@media (max-width: 768px) {
  .url-form {
    flex-direction: column;
  }

  .submit-button {
    width: 100%;
  }

  .header h1 {
    font-size: 2rem;
  }

  .section {
    padding: 15px;
  }

  h2 {
    font-size: 1.5rem;
  }

  h3 {
    font-size: 1.2rem;
  }

  .metric-card {
    padding: 10px;
  }
}

/* Dark Mode Toggle Button */
.dark-mode-toggle {
  background: none;
  border: 2px solid white;
  color: white;
  padding: 8px 16px;
  border-radius: 20px;
  cursor: pointer;
  font-size: 0.9rem;
  margin-top: 10px;
  transition: all 0.3s ease;
}

.dark-mode-toggle:hover {
  background-color: rgba(255, 255, 255, 0.1);
}

.dark-mode .dark-mode-toggle {
  border-color: #bb86fc;
  color: #bb86fc;
}

.dark-mode .dark-mode-toggle:hover {
  background-color: rgba(187, 134, 252, 0.1);
}
</style>