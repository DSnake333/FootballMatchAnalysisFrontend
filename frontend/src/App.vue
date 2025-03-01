<template>
  <!-- Loading Screen -->
  <div v-if="isInitialLoading" class="loading-screen">
    <div class="logo-container">
      <div class="football-icon">
        <i class="fas fa-futbol"></i>
      </div>
      <h1 class="loading-title">Football Data Analysis</h1>
      <p class="loading-subtitle">Loading your football experience...</p>
      <div class="hexagon-loader">
        <div class="hexagon"></div>
        <div class="hexagon"></div>
        <div class="hexagon"></div>
      </div>
      <div class="loading-dots">
        <span></span>
        <span></span>
        <span></span>
      </div>
    </div>
  </div>
  <div v-else :class="{'dark-mode': isDarkMode}">
    <!-- Header (always visible) -->
    <header class="header">
      <div class="header-left">
        <button @click="goHome" class="home-button" aria-label="Go to Home">
          <i class="fas fa-home"></i>
        </button>
        <h1>
          <i class="fas fa-futbol"></i> Football Data Analysis
        </h1>
      </div>
      <button @click="toggleDarkMode" class="dark-mode-toggle">
        <i :class="isDarkMode ? 'fas fa-sun' : 'fas fa-moon'"></i>
        {{ isDarkMode ? '' : '' }}
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
    <div v-else-if="!hasData" class="no-data-container">
      <div class="example-url-box">
        <h3><i class="fas fa-lightbulb"></i> Example URL Format</h3>
        <p class="url-format">https://fbref.com/en/matches/[match-id]/[team-vs-team]</p>

        <div class="analysis-overview">
          <h3><i class="fas fa-chart-bar"></i> Available Analysis</h3>
          <div class="analysis-grid">
            <div class="analysis-item">
              <i class="fas fa-chess-king"></i>
              <h4>Goalkeeper Analysis</h4>
              <p>Save efficiency, cross control, passing effectiveness, and defensive actions</p>
            </div>
            <div class="analysis-item">
              <i class="fas fa-bullseye"></i>
              <h4>Shot Analysis</h4>
              <p>Shot outcomes, xG heatmaps, shooting efficiency, and shot-creating actions</p>
            </div>
            <div class="analysis-item">
              <i class="fas fa-passport"></i>
              <h4>Passing Analysis</h4>
              <p>Pass types, progressive passes, key passes, and team passing networks</p>
            </div>
            <div class="analysis-item">
              <i class="fas fa-shield-alt"></i>
              <h4>Defense Analysis</h4>
              <p>Tackling efficiency, defensive pressure, interceptions, and defensive impact</p>
            </div>
            <div class="analysis-item">
              <i class="fas fa-running"></i>
              <h4>Possession Analysis</h4>
              <p>Ball carries, progressive receptions, touch distribution, and control metrics</p>
            </div>
            <div class="analysis-item">
              <i class="fas fa-chart-pie"></i>
              <h4>Miscellaneous Stats</h4>
              <p>Aerial duels, fouls, defensive actions, and overall player impact</p>
            </div>
          </div>
        </div>

        <div class="url-tips">
          <p><i class="fas fa-info-circle"></i> Note:</p>
          <ul>
            <li>Visit fbref.com and navigate to any match summary page</li>
            <li>Copy the complete URL from your browser's address bar</li>
            <li>Paste it in the input field above</li>
            <li>Some analysis might not be available due to missing stats</li>
          </ul>
        </div>
      </div>
    </div>

    <!-- Interactive Field Section (when data is loaded but no section is selected) -->
    <div v-else-if="hasData && !activeSection" class="interactive-section">
      <h2>Select an area to analyze</h2>
      <InteractiveField @section-selected="showSection" />
    </div>

    <!-- Main Content (only visible when a section is selected) -->
    <main v-else-if="hasData && activeSection" class="main-content">
      <!-- Back button when viewing a section -->
      <button
        @click="activeSection = null"
        class="back-button"
      >
        <i class="fas fa-arrow-left"></i> Back to Field View
      </button>

      <!-- Keeper Stats Analysis Section -->
      <section v-if="activeSection === 'keeper'" id="keeper-analysis" class="section">
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
      <section v-if="activeSection === 'possession'" id="possession-analysis" class="section">
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
      <section v-if="activeSection === 'defense'" id="defense-analysis" class="section">
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
      <section v-if="activeSection === 'shots'" id="shots-analysis" class="section">
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
      <section v-if="activeSection === 'passing'" id="passing-analysis" class="section">
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
      <section v-if="activeSection === 'misc'" id="misc-analysis" class="section">
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
import InteractiveField from "./components/InteractiveField.vue";

export default {
  name: 'App',
  components: {
    InteractiveField,
    KeeperMetrics,
  },
  data() {
    return {
      isInitialLoading: true,
      isDarkMode: false,
      showBackToTop: false,
      matchUrl: "",
      isLoading: false,
      activeSection: null,

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
    goHome() {
      this.activeSection = null;
      this.resetData();
      window.scrollTo({ top: 0, behavior: 'smooth' });
    },
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
      this.activeSection = null;
      this.isLoading = true;
      // Reset all data before new analysis
      this.resetData();

      try {
        const response = await fetch("https://footballmatchanalysisbackend.onrender.com/analyze-match", {
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
    showSection(section) {
      console.log('Showing section:', section); // Add this for debugging
      if (this.hasData) {
        this.activeSection = section;
        window.scrollTo({ top: 0, behavior: 'smooth' });
      } else {
        alert('Please analyze a match first to view statistics.');
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
        const response = await fetch("https://footballmatchanalysisbackend.onrender.com/visualizations/metrics");
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

    // Add this timeout to hide the loading screen after 2.5 seconds
    setTimeout(() => {
      this.isInitialLoading = false;
    }, 2500);
  },
  beforeUnmount() {
    window.removeEventListener("scroll", this.handleScroll);
  },
};
</script>

<style>
/* Global Styles */
@import url('https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0-beta3/css/all.min.css');
@import url('https://fonts.googleapis.com/css2?family=IBM+Plex+Mono:ital,wght@0,100;0,200;0,300;0,400;0,500;0,600;0,700;1,100;1,200;1,300;1,400;1,500;1,600;1,700&family=Major+Mono+Display&display=swap');

/* Reset and Base Styles */
/* Light Mode Colors */
:root {
  --background-light: #f7f4ed;
  --text-primary-light: #242424;
  --text-secondary-light: #6B6B6B;
  --accent-light: #1a8917;
  --border-light: #E6E6E6;
  --input-background-light: #ffffff;
  --card-background-light: #ffffff;
}

/* Dark Mode Colors */
:root {
  --background-dark: #242424;
  --text-primary-dark: #E6E6E6;
  --text-secondary-dark: #A8A8A8;
  --accent-dark: #1a8917;
  --border-dark: #404040;
  --input-background-dark: #1a1a1a;
  --card-background-dark: #1a1a1a;
}

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

html, body {
  margin: 0;
  padding: 0;
  min-height: 100vh;
  width: 100%;
  overflow-x: hidden;
  background-color: var(--background-light);
  color: var(--text-primary-light);
}


#app {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  width: 100%;
  overflow-x: hidden;
  background-color: var(--background-light);
}

body {
  font-family: 'IBM Plex Mono', sans-serif;
  transition: background-color 0.3s, color 0.3s;
  background-color: var(--background-light);
  text-rendering: optimizeLegibility;
  -webkit-font-smoothing: antialiased;
}

.dark-mode {
  background-color: var(--background-dark);
  color: var(--text-primary-dark);
  min-height: 100vh;
}

/* Header Styles */
.header {
  width: 100%;
  padding: 20px 40px; /* Increased horizontal padding */
  background-color: var(--background-light);
  border-bottom: 1px solid var(--border-light);
  display: flex;
  justify-content: space-between; /* Spreads items to opposite ends */
  align-items: center;
  position: sticky; /* Optional: makes header stick to top */
  top: 0;
  z-index: 1000;
}

.dark-mode .header {
  background-color: var(--background-dark);
  border-bottom: 1px solid var(--border-dark);
}

.header h1 {
  color: var(--text-primary-light);
  font-size: 1.5rem;
  margin: 0;
}

.dark-mode .header h1{
  color: var(--text-primary-dark)
}

.header h1 i {
  margin-right: 10px;
}

/* URL Input Container Styles */
.url-input-container {
  width: 100%;
  max-width: 800px;
  margin: 3rem auto;
  padding: 1rem;
  background-color: var(--background-light);
  border-radius: 20px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.1);
}

.dark-mode .url-input-container {
  background-color: var(--background-dark);
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.3);
}

.url-form {
  display: flex;
  gap: 1rem;
  position: relative;
  transition: transform 0.3s ease;
}

.url-form:hover {
  transform: translateY(-2px);
}

.url-input {
  flex: 1;
  padding: 1rem 1.5rem;
  border: 1px solid var(--border-light);
  border-radius: 12px;
  font-size: 1.1rem;
  transition: all 0.3s ease;
  background-color: var(--input-background-light);
  color: var(--text-primary-light);
  box-shadow: inset 0 2px 4px rgba(0, 0, 0, 0.05);
}

.url-input:focus {
  border-color: #34ab45;
  box-shadow: 0 0 0 3px rgba(0, 0, 0, 0.4);
  outline: none;
}

.dark-mode .url-input {
  background-color: var(--input-background-dark);
  border: 1px solid var(--border-dark);
  color: var(--text-primary-dark);
}

.dark-mode .url-input:focus {
  border-color: #34ab45;
  box-shadow: 0 0 0 3px rgba(0, 0, 0, 0.4);
}

.submit-button {
  padding: 1rem 2rem;
  background: var(--accent-light);
  box-shadow: none;
  color: white;
  border: none;
  border-radius: 12px;
  cursor: pointer;
  font-size: 1.1rem;
  font-weight: 500;
  transition: all 0.3s ease;
  white-space: nowrap;
  display: flex;
  align-items: center;
  gap: 8px;
}

.submit-button i {
  font-size: 1.2rem;
}

.submit-button:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.4);
}

.submit-button:active {
  transform: translateY(1px);
}

.dark-mode .submit-button {
  background: var(--accent-dark);
}

.dark-mode .submit-button:hover {
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.4);
}

.submit-button:disabled {
  background: linear-gradient(135deg, #cccccc, #999999);
  cursor: not-allowed;
  transform: none;
  box-shadow: none;
}
.url-input::placeholder {
  color: #999;
  opacity: 0.8;
}

.dark-mode .url-input::placeholder {
  color: #666;
}
/* Loading Spinner and Messages */
.loading-spinner {
  text-align: center;
  margin: 1rem auto;
  max-width: 800px;
  padding: 2rem;
}

.loading-spinner {
  font-size: 1.5rem;
  color: #34ab45;
}

.dark-mode .loading-spinner {
  color: #34ab45;
}

.no-data-message {
  text-align: center;
  padding: 0.75rem; /* Reduced from 2rem */
  background: linear-gradient(145deg, #ffffff, #f0f0f0);
  border-radius: 10px; /* Reduced from 15px */
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
  width: 100%;
  margin-bottom: 1rem; /* Added to create some space between message and example box */
}

.dark-mode .no-data-message {
  background: linear-gradient(145deg, #1a1a1a, #2d2d2d);
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.2);
}

.no-data-message p {
  margin: 0; /* Remove default paragraph margin */
  font-size: 0.9rem; /* Reduced font size */
  color: #666;
  line-height: 1.2; /* Reduced line height */
}

.dark-mode .no-data-message p {
  color: #ccc;
}

/* Main Content and Sections */
.main-content {
  width: 100%;
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
  flex: 1;
  display: flex;
  flex-direction: column;
  min-height: calc(100vh - 200px); /* Adjust based on your header height */
}

.interactive-section {
  width: 100%;
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
  display: flex;
  flex-direction: column;
  align-items: center;
  flex: 1;
  min-height: calc(100vh - 300px); /* Adjust based on your header and input section heights */
}

.section {
  width: 100%;
  margin-bottom: 40px;
  padding: 20px;
  border-radius: 8px;
  background-color: var(--card-background-light);
  box-shadow: 0 1px 4px rgba(0,0,0,0.05);
  transition: transform 0.3s;
}

.section:hover {
  transform: translateY(-5px);
}

.dark-mode .section {
  background-color: var(--card-background-dark);
}

/* Metric Cards */
.metric-card {
  width: 100%;
  margin-bottom: 20px;
  padding: 15px;
  background-color: var(--card-background-light);
  border-radius: 8px;
  transition: transform 0.3s;
}

.metric-card:hover {
  transform: scale(1.02);
}

.dark-mode .metric-card {
  background-color: var(--card-background-dark);
}

/* Typography */
h2 {
  font-size: 2rem;
  color: var(--text-primary-light);
  margin-bottom: 1rem;
  text-align: center;
}

.dark-mode h2 {
  color: var(--text-primary-dark);
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
  color: var(--text-secondary-light);
  line-height: 1.6;
}

.dark-mode p {
  color: var(--text-secondary-dark);
}

/* Buttons */
.back-button {
  align-self: flex-start;
  margin-bottom: 20px;
  padding: 10px 20px;
  background-color: #34ab45;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: all 0.3s ease;
  display: inline-flex;
  align-items: center;
  gap: 8px;
}

.back-button:hover {
  background-color: rgba(0, 0, 0, 0.4);
  transform: translateX(-5px);
}

.dark-mode .back-button {
  background-color: #34ab45;
}

.back-to-top {
  position: fixed;
  bottom: 20px;
  right: 20px;
  background-color: #34ab45;
  color: white;
  border: none;
  border-radius: 50%;
  width: 50px;
  height: 50px;
  font-size: 1.2rem;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  z-index: 1000;
  display: flex;
  align-items: center;
  justify-content: center;
}

.back-to-top:hover {
  background-color: #34ab45;
  transform: scale(1.1);
}

.dark-mode .back-to-top {
  background-color: #34ab45;
}

/* Dark Mode Toggle */
.dark-mode-toggle {
  background: none;
  border: 1px solid var(--border-light);
  color: var(--text-secondary-light);
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
  border: 1px solid var(--border-dark);
  color: #ffffff;
}

.dark-mode .dark-mode-toggle:hover {
  background-color: rgba(187, 134, 252, 0.1);
}



/* Responsive Design */
@media (max-width: 768px) {
  .url-form {
    flex-direction: column;
  }

  .url-input-container {
    padding: 1.5rem;
    margin: 1rem auto;
  }

  .url-input,
  .submit-button {
    width: 100%;
    padding: 0.875rem 1.25rem;
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

  .main-content,
  .interactive-section {
    padding: 15px;
  }

  .back-to-top {
    width: 40px;
    height: 40px;
    font-size: 1rem;
    bottom: 15px;
    right: 15px;
  }
}

@media (max-width: 480px) {
  .header h1 {
    font-size: 1.5rem;
  }

  .url-input-container {
    padding: 0 15px;
  }

  .no-data-message {
    padding: 1.5rem;
    margin: 1.5rem auto;
  }
}
.no-data-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1rem;
  margin: auto;
  max-width: 1200px;
  padding: 0 30px;
}

.no-data-message {
  text-align: center;
  padding: 1rem;
  background-color: var(--background-light);
  border-radius: 15px;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.05);
  width: 100%;
}

.dark-mode .no-data-message {
  background-color: var(--background-dark);
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.2);
}

.example-url-box {
  background-color: var(--card-background-light);
  border-radius: 15px;
  display: grid;
  grid-gap: 1rem;
  padding-top: 1.5rem ;
  padding-left: 1.5rem ;
  padding-right: 1.5rem ;
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.08);
  border: 1px solid rgba(0, 123, 255, 0.1);
  transition: all 0.3s ease;
}

.dark-mode .example-url-box {
  background-color: var(--card-background-dark);
  border: 1px solid rgba(187, 134, 252, 0.1);
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.3);
}

.example-url-box:hover {
  transform: translateY(-3px);
  box-shadow: 0 8px 25px rgba(0, 0, 0, 0.12);
}

.example-url-box h3 {
  color: #34ab45;
  font-size: 1rem;
  margin-bottom: 0.5rem;
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.dark-mode .example-url-box h3 {
  color: #34ab45;
}

.url-format {
  background: rgba(0, 123, 255, 0.1);
  padding: 0.75rem;
  border-radius: 8px;
  font-family: monospace;
  font-size: 0.85rem;
  color: #0056b3;
  margin: 0.5rem 0;
  word-break: break-all;
}

.dark-mode .url-format {
  background: rgba(187, 134, 252, 0.1);
  color: #bb86fc;
}

.url-tips {
  margin-top: 1rem;
  padding-top: 1rem;
  border-top: 1px solid rgba(0, 0, 0, 0.1);
}

.dark-mode .url-tips {
  border-top-color: rgba(255, 255, 255, 0.1);
}

.url-tips p {
  color: #666;
  font-weight: 500;
  display: flex;
  align-items: center;
  gap: 0.5rem;
  margin-bottom: 0.25rem;
  font-size: 0.9rem;
}

.dark-mode .url-tips p {
  color: #ccc;
}

.url-tips ul {
  list-style: none;
  padding-left: 1rem;
}

.url-tips li {
  color: #666;
  margin: 0.25rem 0;
  font-size: 0.8rem;
  position: relative;
}

.dark-mode .url-tips li {
  color: #ccc;
}

.url-tips li::before {
  content: "•";
  color: #007bff;
  position: absolute;
  left: -1rem;
}

.dark-mode .url-tips li::before {
  color: #bb86fc;
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

/* Responsive adjustments */
@media (max-width: 768px) {
  .no-data-container {
    margin: 1rem auto;
    padding: 0 15px;
  }

  .example-url-box {
    padding: 1.5rem;
  }

  .url-format {
    font-size: 0.8rem;
    padding: 0.75rem;
  }

  .url-tips {
    margin-top: 1rem;
    padding-top: 1rem;
  }

  .url-tips li {
    font-size: 0.9rem;
  }
}

.analysis-overview {
  margin-top: 1rem;
  padding-top: 1rem;
  border-top: 1px solid rgba(0, 0, 0, 0.1);
}

.dark-mode .analysis-overview {
  border-top-color: rgba(255, 255, 255, 0.1);
}

.analysis-overview h3 {
  margin-bottom: 1.5rem;
}

.analysis-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 1rem;
  margin-bottom: 2rem;
}

.analysis-item {
  background: rgba(255, 255, 255, 0.5);
  padding: 1rem;
  border-radius: 12px;
  transition: all 0.3s ease;
  border: 1px solid rgba(0, 123, 255, 0.1);
}

.dark-mode .analysis-item {
  background: rgba(45, 45, 45, 0.5);
  border-color: rgba(187, 134, 252, 0.1);
}

.analysis-item:hover {
  transform: translateY(-3px);
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
}

.dark-mode .analysis-item:hover {
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.3);
}

.analysis-item i {
  font-size: 1.2rem;
  color: #34ab45;
  margin-bottom: 0.5rem;
}

.dark-mode .analysis-item i {
  color: #34ab45;
}

.analysis-item h4 {
  font-size: 1rem;
  color: #333;
  margin-bottom: 0.25rem;
}

.dark-mode .analysis-item h4 {
  color: #ffffff;
}

.analysis-item p {
  font-size: 0.7rem;
  color: #666;
  line-height: 1.2;
  margin: 0;
}

.dark-mode .analysis-item p {
  color: #ccc;
}

/* Responsive adjustments */

@media (max-width: 1200px) {
  .analysis-grid {
    grid-template-columns: repeat(2, 1fr);
  }
}

@media (max-width: 768px) {
  .analysis-grid {
    grid-template-columns: 1fr;
    gap: 1rem;
  }

  .analysis-item {
    padding: 1rem;
  }

  .analysis-item h4 {
    font-size: 1rem;
  }

  .analysis-item p {
    font-size: 0.85rem;
  }

  .analysis-overview {
    margin-top: 1.5rem;
    padding-top: 1.5rem;
  }

  .no-data-container {
    padding: 0 15px;
  }
}
/* Minimal Dark Loading Screen with Advanced Animation */
.loading-screen {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: #121212;
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 9999;
  overflow: hidden;
}

.dark-mode .loading-screen {
  background: #000000;
}

.logo-container {
  position: relative;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 3rem;
  border-radius: 0.5rem;
  background-color: rgba(24, 24, 24, 0.7);
  transition: all 0.3s ease;
}

.dark-mode .logo-container {
  background-color: rgba(18, 18, 18, 0.7);
}

.football-icon {
  position: relative;
  font-size: 4rem;
  color: #ffffff;
  animation: float 3s ease-in-out infinite;
  z-index: 2;
}

.football-icon::after {
  content: '';
  position: absolute;
  top: 50%;
  left: 50%;
  width: 60px;
  height: 60px;
  border-radius: 50%;
  background-color: rgba(255, 255, 255, 0.05);
  transform: translate(-50%, -50%);
  z-index: -1;
  animation: pulse 2s infinite;
}

.loading-title {
  margin-top: 2rem;
  font-size: 2rem;
  font-weight: 600;
  color: #ffffff;
  letter-spacing: 1px;
  opacity: 0;
  animation: fadeInUp 1s ease-out forwards 0.5s;
}

.loading-subtitle {
  margin-top: 0.5rem;
  font-size: 0.9rem;
  font-weight: 400;
  color: #888888;
  opacity: 0;
  animation: fadeInUp 1s ease-out forwards 0.7s;
}

/* New circular loader animation */
.loader-ring {
  margin-top: 2rem;
  display: inline-block;
  position: relative;
  width: 50px;
  height: 50px;
}

.loader-ring div {
  box-sizing: border-box;
  display: block;
  position: absolute;
  width: 40px;
  height: 40px;
  margin: 5px;
  border: 2px solid #ffffff;
  border-radius: 50%;
  animation: loader-ring 1.2s cubic-bezier(0.5, 0, 0.5, 1) infinite;
  border-color: #ffffff transparent transparent transparent;
}

.loader-ring div:nth-child(1) {
  animation-delay: -0.45s;
}

.loader-ring div:nth-child(2) {
  animation-delay: -0.3s;
}

.loader-ring div:nth-child(3) {
  animation-delay: -0.15s;
}

/* Alternative hexagonal loader */
.hexagon-loader {
  margin-top: 2rem;
  position: relative;
  width: 60px;
  height: 60px;
  background: transparent;
}

.hexagon-loader .hexagon {
  position: absolute;
  width: 100%;
  height: 100%;
  border: 2px solid transparent;
}

.hexagon-loader .hexagon:nth-child(1) {
  border-top-color: #ffffff;
  border-bottom-color: #ffffff;
  animation: hexagon-rotate 1.5s linear infinite;
}

.hexagon-loader .hexagon:nth-child(2) {
  border-left-color: #ffffff;
  border-right-color: #ffffff;
  animation: hexagon-rotate 2s linear infinite reverse;
}

.hexagon-loader .hexagon:nth-child(3) {
  border: 2px dotted rgba(255, 255, 255, 0.3);
  animation: hexagon-rotate 3s linear infinite;
}

.hexagon {
  clip-path: polygon(50% 0%, 100% 25%, 100% 75%, 50% 100%, 0% 75%, 0% 25%);
}

/* Remove the old loading dots */
.loading-dots {
  display: none;
}

/* Animations */
@keyframes float {
  0%, 100% {
    transform: translateY(0);
  }
  50% {
    transform: translateY(-10px);
  }
}

@keyframes pulse {
  0% {
    transform: translate(-50%, -50%) scale(1);
    opacity: 0.3;
  }
  100% {
    transform: translate(-50%, -50%) scale(1.5);
    opacity: 0;
  }
}

@keyframes fadeInUp {
  0% {
    opacity: 0;
    transform: translateY(10px);
  }
  100% {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes loader-ring {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}

@keyframes hexagon-rotate {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}

/* Responsive adjustments */
@media (max-width: 768px) {
  .logo-container {
    padding: 2rem;
  }

  .football-icon {
    font-size: 3.5rem;
  }

  .loading-title {
    font-size: 1.75rem;
  }

  .hexagon-loader {
    width: 50px;
    height: 50px;
  }
}
/* Home Button Styles */
.header-left {
  display: flex;
  align-items: center;
  gap: 15px;
}

.home-button {
  background: none;
  border: none;
  color: var(--text-primary-light);
  font-size: 1.2rem;
  cursor: pointer;
  padding: 8px;
  border-radius: 50%;
  transition: all 0.3s ease;
  display: flex;
  align-items: center;
  justify-content: center;
}

.home-button:hover {
  background-color: rgba(0, 0, 0, 0.05);
  transform: scale(1.1);
}

.dark-mode .home-button {
  color: var(--text-primary-dark);
}

.dark-mode .home-button:hover {
  background-color: rgba(255, 255, 255, 0.1);
}
</style>
