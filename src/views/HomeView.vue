<template>
  <main
    class="min-h-screen bg-gradient-to-br from-slate-900 via-purple-900 to-slate-900 text-white font-sans flex items-center justify-center p-4 relative overflow-hidden"
  >
    <div class="absolute inset-0 pointer-events-none">
      <svg class="absolute inset-0 w-full h-full">
        <defs>
          <filter id="glow" x="-50%" y="-50%" width="200%" height="200%">
            <feGaussianBlur stdDeviation="2" result="coloredBlur" />
            <feMerge>
              <feMergeNode in="coloredBlur" />
              <feMergeNode in="SourceGraphic" />
            </feMerge>
          </filter>
          <filter id="strongGlow" x="-50%" y="-50%" width="200%" height="200%">
            <feGaussianBlur stdDeviation="3" result="coloredBlur" />
            <feMerge>
              <feMergeNode in="coloredBlur" />
              <feMergeNode in="SourceGraphic" />
            </feMerge>
          </filter>
        </defs>

        <line
          v-for="connection in connections"
          :key="`${connection.from}-${connection.to}`"
          :x1="particles[connection.from].x + 4"
          :y1="particles[connection.from].y + 4"
          :x2="particles[connection.to].x + 4"
          :y2="particles[connection.to].y + 4"
          :stroke="`rgba(147, 197, 253, ${connection.opacity})`"
          :stroke-width="Math.max(0.5, 2 - connection.distance / 100)"
          class="transition-all duration-300 ease-out"
          filter="url(#glow)"
        />
      </svg>

      <div
        v-for="(particle, index) in particles"
        :key="index"
        class="absolute w-2 h-2 bg-blue-400 rounded-full particle-glow"
        :style="{
          left: particle.x + 'px',
          top: particle.y + 'px',
          opacity: 0.8
        }"
      />
    </div>

    <div class="max-w-2xl mx-auto text-center space-y-8 relative z-10">
      <div class="space-y-4">
        <h1
          class="text-5xl md:text-6xl font-bold bg-gradient-to-r from-blue-400 via-purple-400 to-pink-400 bg-clip-text text-transparent leading-tight entry-title"
        >
          Haii, I'm Averi
        </h1>
        <p class="text-xl md:text-2xl text-slate-300 font-light entry-subtitle">I write bad code</p>
      </div>

      <div class="space-y-2 entry-social-section">
        <!-- <h2 class="text-2xl font-semibold text-slate-200">Connect with me</h2> -->
        <div class="grid grid-cols-2 md:grid-cols-4 gap-4">
          <a
            v-for="(social, index) in socials"
            :key="social.name"
            :href="social.url"
            target="_blank"
            rel="noopener noreferrer"
            class="group flex flex-col items-center p-4 rounded-xl bg-white/5 backdrop-blur-sm border border-white/10 hover:bg-white/10 hover:border-white/20 transition-all duration-300 hover:scale-105 hover:shadow-lg entry-social-card"
            :style="{ 'animation-delay': `${0.5 + index * 0.1}s` }"
          >
            <component :is="social.icon" class="w-8 h-8 mb-2 transition-colors duration-300" :class="social.color" />
            <span class="text-sm font-medium text-slate-300 group-hover:text-white transition-colors duration-300">
              {{ social.name }}
            </span>
          </a>
        </div>
      </div>
    </div>
  </main>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue';
import type { IconType } from 'vue-icons-plus';
import { BsDiscord, BsGithub, BsTwitter } from 'vue-icons-plus/bs';
import { SiBluesky } from 'vue-icons-plus/si';

const socials = [
  {
    name: 'GitHub',
    url: 'https://github.com/averithefox',
    icon: BsGithub,
    color: 'group-hover:text-gray-300'
  },
  {
    name: 'Discord',
    url: 'https://discord.com/users/719890634294427669',
    icon: BsDiscord,
    color: 'group-hover:text-indigo-400'
  },
  {
    name: 'Twitter',
    url: 'https://x.com/averithefox',
    icon: BsTwitter,
    color: 'group-hover:text-sky-400'
  },
  {
    name: 'Bluesky',
    url: 'https://bsky.app/profile/averi.me',
    icon: SiBluesky,
    color: 'group-hover:text-blue-400'
  }
] satisfies { name: string; url: string; icon: IconType; color: string }[];

interface Particle {
  x: number;
  y: number;
  vx: number;
  vy: number;
}

interface Connection {
  from: number;
  to: number;
  distance: number;
  opacity: number;
  targetOpacity: number;
}

const particles = ref<Particle[]>([]);
const connections = ref<Connection[]>([]);
const connectionMap = ref<Map<string, Connection>>(new Map());
const animationId = ref<number | null>(null);

const PARTICLE_COUNT = 25;
const CONNECTION_DISTANCE = 120;
const PARTICLE_SPEED = 0.3;
const FADE_SPEED = 0.05;

const initializeParticles = () => {
  particles.value = [];
  for (let i = 0; i < PARTICLE_COUNT; i++) {
    particles.value.push({
      x: Math.random() * window.innerWidth,
      y: Math.random() * window.innerHeight,
      vx: (Math.random() - 0.5) * PARTICLE_SPEED * 2,
      vy: (Math.random() - 0.5) * PARTICLE_SPEED * 2
    });
  }
};

const calculateDistance = (p1: Particle, p2: Particle): number => {
  const dx = p1.x - p2.x;
  const dy = p1.y - p2.y;
  return Math.sqrt(dx * dx + dy * dy);
};

const getConnectionKey = (from: number, to: number): string => {
  return `${Math.min(from, to)}-${Math.max(from, to)}`;
};

const updateConnections = () => {
  const activeConnections = new Set<string>();

  for (let i = 0; i < particles.value.length; i++) {
    for (let j = i + 1; j < particles.value.length; j++) {
      const distance = calculateDistance(particles.value[i], particles.value[j]);
      const key = getConnectionKey(i, j);

      if (distance < CONNECTION_DISTANCE) {
        activeConnections.add(key);

        if (!connectionMap.value.has(key)) {
          const newConnection: Connection = {
            from: i,
            to: j,
            distance,
            opacity: 0,
            targetOpacity: Math.max(0.1, 0.4 - distance / 300)
          };
          connectionMap.value.set(key, newConnection);
        } else {
          const connection = connectionMap.value.get(key)!;
          connection.distance = distance;
          connection.targetOpacity = Math.max(0.1, 0.4 - distance / 300);
        }
      }
    }
  }

  const connectionsToRemove: string[] = [];

  connectionMap.value.forEach((connection, key) => {
    if (activeConnections.has(key)) {
      if (connection.opacity < connection.targetOpacity) {
        connection.opacity = Math.min(connection.targetOpacity, connection.opacity + FADE_SPEED);
      } else if (connection.opacity > connection.targetOpacity) {
        connection.opacity = Math.max(connection.targetOpacity, connection.opacity - FADE_SPEED);
      }
    } else {
      connection.opacity = Math.max(0, connection.opacity - FADE_SPEED);
      if (connection.opacity <= 0) {
        connectionsToRemove.push(key);
      }
    }
  });

  connectionsToRemove.forEach(key => {
    connectionMap.value.delete(key);
  });

  connections.value = Array.from(connectionMap.value.values());
};

const updateParticles = () => {
  particles.value.forEach(particle => {
    particle.x += particle.vx;
    particle.y += particle.vy;

    if (particle.x <= 0 || particle.x >= window.innerWidth) {
      particle.vx *= -1;
      particle.x = Math.max(0, Math.min(window.innerWidth, particle.x));
    }
    if (particle.y <= 0 || particle.y >= window.innerHeight) {
      particle.vy *= -1;
      particle.y = Math.max(0, Math.min(window.innerHeight, particle.y));
    }
  });
};

const animate = () => {
  updateParticles();
  updateConnections();
  animationId.value = requestAnimationFrame(animate);
};

const handleResize = () => {
  initializeParticles();
  connectionMap.value.clear();
  connections.value = [];
};

onMounted(() => {
  initializeParticles();
  animate();
  window.addEventListener('resize', handleResize);
});

onUnmounted(() => {
  if (animationId.value) {
    cancelAnimationFrame(animationId.value);
  }
  window.removeEventListener('resize', handleResize);
});
</script>

<style scoped>
.particle-glow {
  box-shadow: 0 0 6px rgba(147, 197, 253, 0.6), 0 0 12px rgba(147, 197, 253, 0.4), 0 0 18px rgba(147, 197, 253, 0.2);
  filter: drop-shadow(0 0 4px rgba(147, 197, 253, 0.8));
}

.particle-glow {
  animation: pulse-glow 3s ease-in-out infinite;
}

@keyframes pulse-glow {
  0%,
  100% {
    box-shadow: 0 0 6px rgba(147, 197, 253, 0.6), 0 0 12px rgba(147, 197, 253, 0.4), 0 0 18px rgba(147, 197, 253, 0.2);
    filter: drop-shadow(0 0 4px rgba(147, 197, 253, 0.8));
  }
  50% {
    box-shadow: 0 0 8px rgba(147, 197, 253, 0.8), 0 0 16px rgba(147, 197, 253, 0.6), 0 0 24px rgba(147, 197, 253, 0.4);
    filter: drop-shadow(0 0 6px rgba(147, 197, 253, 1));
  }
}

.entry-title {
  opacity: 0;
  transform: translateY(-30px);
  animation: slideInFromTop 0.8s ease-out 0.2s forwards;
}

.entry-subtitle {
  opacity: 0;
  transform: translateY(20px);
  animation: slideInFromBottom 0.8s ease-out 0.4s forwards;
}

.entry-social-section {
  opacity: 0;
  transform: translateY(30px) scale(0.95);
  animation: slideInWithScale 0.8s ease-out 0.6s forwards;
}

.entry-social-card {
  opacity: 0;
  transform: translateY(20px) scale(0.9);
  animation: slideInCard 0.6s ease-out forwards;
}

@keyframes slideInFromTop {
  from {
    opacity: 0;
    transform: translateY(-30px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes slideInFromBottom {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes slideInWithScale {
  from {
    opacity: 0;
    transform: translateY(30px) scale(0.95);
  }
  to {
    opacity: 1;
    transform: translateY(0) scale(1);
  }
}

@keyframes slideInCard {
  from {
    opacity: 0;
    transform: translateY(20px) scale(0.9);
  }
  to {
    opacity: 1;
    transform: translateY(0) scale(1);
  }
}

@media (prefers-reduced-motion: reduce) {
  .entry-title,
  .entry-subtitle,
  .entry-social-section,
  .entry-social-card {
    animation: fadeInOnly 0.5s ease-out forwards;
  }
}

@keyframes fadeInOnly {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}
</style>
