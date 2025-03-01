<template>
  <div class="analysis-hub">
    <div class="center-circle">
      <div class="inner-circle">
        <span>Match Analysis</span>
      </div>
    </div>

    <div class="analysis-sections">
      <div
        v-for="(section, index) in sections"
        :key="section.id"
        class="analysis-section"
        :style="getSectionStyle(index)"
        @mouseover="showTooltip(section.id)"
        @mouseleave="hideTooltip"
        @click="selectSection(section.id)"
      >
        <div class="section-content">
          <div class="icon-container">
            <i :class="section.icon"></i>
          </div>
          <span class="section-label">{{ section.label }}</span>
        </div>
        <div v-show="activeTooltip === section.id" class="tooltip" :class="getTooltipPosition(index)">
          {{ section.tooltip }}
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'InteractiveField',
  data() {
    return {
      activeTooltip: null,
      activeSection: null,
      sections: [
        {
          id: 'keeper',
          label: 'Keeper Analysis',
          icon: 'fas fa-hands',
          tooltip: 'Analyze goalkeeper performance'
        },
        {
          id: 'shots',
          label: 'Shots Analysis',
          icon: 'fas fa-bullseye',
          tooltip: 'Explore shot patterns'
        },
        {
          id: 'possession',
          label: 'Possession',
          icon: 'fas fa-futbol',
          tooltip: 'Review ball control'
        },
        {
          id: 'passing',
          label: 'Passing Analysis',
          icon: 'fas fa-random',
          tooltip: 'Examine passing patterns'
        },
        {
          id: 'misc',
          label: 'Misc Analysis',
          icon: 'fas fa-chart-pie',
          tooltip: 'View other statistics'
        }
      ]
    }
  },
  mounted() {
    window.addEventListener('resize', this.handleResize);
  },
  beforeUnmount() {
    window.removeEventListener('resize', this.handleResize);
  },
  methods: {
    showTooltip(id) {
      this.activeTooltip = id;
    },
    hideTooltip() {
      this.activeTooltip = null;
    },
    selectSection(id) {
      this.activeSection = id;
      this.$emit('section-selected', id);
    },
    handleResize() {
      this.$forceUpdate();
    },
    getSectionStyle(index) {
      const totalSections = this.sections.length;
      const angle = (270 + (index * (360 / totalSections))) * (Math.PI / 180);

      let radius = 150; // default radius
      if (window.innerWidth <= 668) {
        radius = 130;
      }
      if (window.innerWidth <= 520) {
        radius = 110;
      }

      const x = Math.cos(angle) * radius;
      const y = Math.sin(angle) * radius;

      return {
        left: `calc(50% + ${x}px)`,
        top: `calc(50% + ${y}px)`,
        transform: 'translate(-50%, -50%)'
      };
    },
    getTooltipPosition(index) {
      const positions = ['top', 'right', 'bottom', 'left', 'top'];
      return positions[index];
    }
  }
}
</script>

<style scoped>

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

@keyframes pulse {
  0%, 100% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.05);
  }
}

.analysis-hub {
  position: relative;
  width: 500px;
  height: 500px;
  margin: 40px auto;
  display: flex;
  justify-content: center;
  align-items: center;
  overflow: visible;
  background: radial-gradient(circle, var(--background-light) 0%, #f0ebe1 100%);
  border-radius: 50%;
  box-shadow: 0 0 30px rgba(0, 0, 0, 0.1);
}

.center-circle {
  position: absolute;
  width: 160px;
  height: 160px;
  background: rgba(255, 255, 255, 0.5);
  border: 2px solid var(--accent-light);
  border-radius: 50%;
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 2;
  padding: 15px;
  box-shadow: 0 0 15px rgba(26, 137, 23, 0.2);
}

.inner-circle {
  width: 100%;
  height: 100%;
  background: var(--accent-light);
  color: white;
  border-radius: 50%;
  display: flex;
  justify-content: center;
  align-items: center;
  font-weight: 600;
  font-size: 1.1rem;
  padding: 15px;
  box-shadow: inset 0 0 10px rgba(0, 0, 0, 0.1);
  animation: pulse 2s infinite;
}

.inner-circle span {
  text-align: center;
  padding: 10px;
}

.analysis-sections {
  position: absolute;
  width: 100%;
  height: 100%;
}

.analysis-section {
  position: absolute;
  width: 110px;
  height: 110px;
  transition: transform 0.3s ease, filter 0.3s ease;
}

.section-content {
  position: relative;
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  background: rgba(255, 255, 255, 0.8);
  border: 1px solid var(--border-light);
  border-radius: 50%;
  padding: 10px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.05);
  transition: transform 0.3s ease, box-shadow 0.3s ease;
}

.section-content:hover {
  transform: scale(1.05);
  box-shadow: 0 0 15px rgba(0, 0, 0, 0.1);
}

.icon-container {
  font-size: 1.6rem;
  color: var(--accent-light);
  margin-bottom: 8px;
}

.section-label {
  font-size: 0.8rem;
  text-align: center;
  font-weight: 500;
  color: var(--text-primary-light);
}

.tooltip {
  position: absolute;
  background: var(--accent-light);
  color: white;
  padding: 8px 12px;
  border-radius: 8px;
  font-size: 0.85rem;
  white-space: nowrap;
  z-index: 1000;
  max-width: 200px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
  pointer-events: none;
}

.tooltip.top {
  bottom: 100%;
  left: 50%;
  transform: translateX(-50%);
  margin-bottom: 10px;
}

.tooltip.right {
  left: 100%;
  top: 50%;
  transform: translateY(-50%);
  margin-left: 10px;
}

.tooltip.bottom {
  top: 100%;
  left: 50%;
  transform: translateX(-50%);
  margin-top: 10px;
}

.tooltip.left {
  right: 100%;
  top: 50%;
  transform: translateY(-50%);
  margin-right: 10px;
}

.dark-mode .analysis-hub {
  background: radial-gradient(circle, var(--background-dark) 0%, #1a1a1a 100%);
}

.dark-mode .inner-circle {
  background: var(--accent-dark);
}

.dark-mode .icon-container {
  color: var(--accent-dark);
}

.dark-mode .section-label {
  color: var(--text-primary-dark);
}

.dark-mode .section-content {
  background: rgba(45, 45, 45, 0.8);
  border: 1px solid var(--border-dark);
}

.dark-mode .center-circle {
  border: 3px solid limegreen;
}

.dark-mode .tooltip {
  background: var(--accent-dark);
}

/* Responsive styles */
@media (max-width: 868px) {
  .analysis-hub {
    width: 500px;
    height: 500px;
  }

  .center-circle {
    width: 140px;
    height: 140px;
  }

  .inner-circle {
    width: 120px;
    height: 120px;
  }

  .analysis-section {
    width: 100px;
    height: 100px;
  }

  .section-label {
    font-size: 0.75rem;
  }
}

@media (max-width: 668px) {
  .analysis-hub {
    width: 400px;
    height: 400px;
  }

  .center-circle {
    width: 130px;
    height: 130px;
  }

  .inner-circle {
    width: 110px;
    height: 110px;
  }

  .analysis-section {
    width: 90px;
    height: 90px;
  }
}

@media (max-width: 520px) {
  .analysis-hub {
    width: 350px;
    height: 350px;
  }

  .center-circle {
    width: 120px;
    height: 120px;
  }

  .inner-circle {
    width: 100px;
    height: 100px;
  }

  .analysis-section {
    width: 85px;
    height: 85px;
  }

  .icon-container {
    font-size: 1.3rem;
  }

  .section-label {
    font-size: 0.7rem;
  }
}

@media (max-width: 420px) {
  .analysis-hub {
    transform: scale(0.9);
  }
}

@media (max-width: 375px) {
  .analysis-hub {
    transform: scale(0.8);
  }
}
</style>