<template>
  <main
    class="min-h-screen bg-gradient-to-br from-slate-900 via-purple-900 to-slate-900 text-white font-sans flex items-center justify-center p-4 relative overflow-hidden"
  >
    <canvas
      ref="canvasRef"
      class="absolute inset-0 w-full h-full pointer-events-none"
      :width="canvasWidth"
      :height="canvasHeight"
    />

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
import { ref, onMounted, onUnmounted, nextTick } from 'vue';
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

const canvasRef = ref<HTMLCanvasElement | null>(null);
const canvasWidth = ref(0);
const canvasHeight = ref(0);
const particles = ref<Particle[]>([]);
const connections = ref<Connection[]>([]);
const connectionMap = ref<Map<string, Connection>>(new Map());
const animationId = ref<number | null>(null);

const PARTICLE_COUNT = 100;
const CONNECTION_DISTANCE = 150;
const PARTICLE_SPEED = 0.8;
const FADE_SPEED = 0.08;
const PARTICLE_SIZE = 3;
const MAX_LINE_WIDTH = 2.5;
const COLLISION_DISTANCE = 12;
const REPULSION_FORCE = 0.02;
const CONNECTION_BOOST = 0.003;
const MIN_VELOCITY = 0.2;

const updateCanvasSize = () => {
  canvasWidth.value = window.innerWidth;
  canvasHeight.value = window.innerHeight;
};

const initializeParticles = () => {
  particles.value = [];
  for (let i = 0; i < PARTICLE_COUNT; i++) {
    particles.value.push({
      x: Math.random() * canvasWidth.value,
      y: Math.random() * canvasHeight.value,
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
            targetOpacity: Math.max(0.1, 0.6 - distance / 250)
          };
          connectionMap.value.set(key, newConnection);
        } else {
          const connection = connectionMap.value.get(key)!;
          connection.distance = distance;
          connection.targetOpacity = Math.max(0.1, 0.6 - distance / 250);
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
  connections.value.forEach(connection => {
    if (connection.opacity <= 0) return;

    const fromParticle = particles.value[connection.from];
    const toParticle = particles.value[connection.to];

    const dx = toParticle.x - fromParticle.x;
    const dy = toParticle.y - fromParticle.y;
    const distance = Math.sqrt(dx * dx + dy * dy);

    if (distance > 0) {
      const normalX = dx / distance;
      const normalY = dy / distance;

      const fromDotProduct = fromParticle.vx * normalX + fromParticle.vy * normalY;
      const toDotProduct = toParticle.vx * normalX + toParticle.vy * normalY;

      const boostStrength = CONNECTION_BOOST * connection.opacity;

      if (fromDotProduct > 0) {
        fromParticle.vx += normalX * boostStrength * Math.abs(fromDotProduct);
        fromParticle.vy += normalY * boostStrength * Math.abs(fromDotProduct);
      } else if (fromDotProduct < 0) {
        fromParticle.vx -= normalX * boostStrength * Math.abs(fromDotProduct);
        fromParticle.vy -= normalY * boostStrength * Math.abs(fromDotProduct);
      }

      if (toDotProduct > 0) {
        toParticle.vx += normalX * boostStrength * Math.abs(toDotProduct);
        toParticle.vy += normalY * boostStrength * Math.abs(toDotProduct);
      } else if (toDotProduct < 0) {
        toParticle.vx -= normalX * boostStrength * Math.abs(toDotProduct);
        toParticle.vy -= normalY * boostStrength * Math.abs(toDotProduct);
      }
    }
  });

  for (let i = 0; i < particles.value.length; i++) {
    for (let j = i + 1; j < particles.value.length; j++) {
      const p1 = particles.value[i];
      const p2 = particles.value[j];

      const dx = p2.x - p1.x;
      const dy = p2.y - p1.y;
      const distance = Math.sqrt(dx * dx + dy * dy);

      if (distance < COLLISION_DISTANCE && distance > 0) {
        const normalX = dx / distance;
        const normalY = dy / distance;

        const relativeVx = p2.vx - p1.vx;
        const relativeVy = p2.vy - p1.vy;

        const relativeSpeed = relativeVx * normalX + relativeVy * normalY;

        if (relativeSpeed < 0) {
          const restitution = 0.8;
          const impulse = relativeSpeed * restitution;

          p1.vx += impulse * normalX;
          p1.vy += impulse * normalY;
          p2.vx -= impulse * normalX;
          p2.vy -= impulse * normalY;
        }

        const overlap = COLLISION_DISTANCE - distance;
        const repulsionX = normalX * overlap * REPULSION_FORCE;
        const repulsionY = normalY * overlap * REPULSION_FORCE;

        p1.vx -= repulsionX;
        p1.vy -= repulsionY;
        p2.vx += repulsionX;
        p2.vy += repulsionY;
      }
    }
  }

  particles.value.forEach(particle => {
    particle.x += particle.vx;
    particle.y += particle.vy;

    particle.vx *= 0.9995;
    particle.vy *= 0.9995;

    const currentSpeed = Math.sqrt(particle.vx * particle.vx + particle.vy * particle.vy);
    if (currentSpeed < MIN_VELOCITY) {
      const angle = Math.random() * Math.PI * 2;
      const boost = MIN_VELOCITY - currentSpeed + 0.1;
      particle.vx += Math.cos(angle) * boost;
      particle.vy += Math.sin(angle) * boost;
    }

    const maxVelocity = 4;
    const speed = Math.sqrt(particle.vx * particle.vx + particle.vy * particle.vy);
    if (speed > maxVelocity) {
      particle.vx = (particle.vx / speed) * maxVelocity;
      particle.vy = (particle.vy / speed) * maxVelocity;
    }

    if (particle.x <= 0 || particle.x >= canvasWidth.value) {
      particle.vx *= -0.9;
      particle.x = Math.max(0, Math.min(canvasWidth.value, particle.x));
    }
    if (particle.y <= 0 || particle.y >= canvasHeight.value) {
      particle.vy *= -0.9;
      particle.y = Math.max(0, Math.min(canvasHeight.value, particle.y));
    }
  });
};

const drawParticles = (ctx: CanvasRenderingContext2D) => {
  particles.value.forEach(particle => {
    ctx.save();
    ctx.beginPath();

    const gradient = ctx.createRadialGradient(particle.x, particle.y, 0, particle.x, particle.y, PARTICLE_SIZE * 2);
    gradient.addColorStop(0, 'rgba(147, 197, 253, 0.8)');
    gradient.addColorStop(0.3, 'rgba(147, 197, 253, 0.4)');
    gradient.addColorStop(0.6, 'rgba(147, 197, 253, 0.2)');
    gradient.addColorStop(1, 'rgba(147, 197, 253, 0)');

    ctx.fillStyle = gradient;
    ctx.arc(particle.x, particle.y, PARTICLE_SIZE * 2, 0, Math.PI * 2);
    ctx.fill();

    ctx.beginPath();
    ctx.fillStyle = 'rgba(147, 197, 253, 0.9)';
    ctx.arc(particle.x, particle.y, PARTICLE_SIZE, 0, Math.PI * 2);
    ctx.fill();

    ctx.restore();
  });
};

const drawConnections = (ctx: CanvasRenderingContext2D) => {
  connections.value.forEach(connection => {
    if (connection.opacity <= 0) return;

    const fromParticle = particles.value[connection.from];
    const toParticle = particles.value[connection.to];

    ctx.save();

    const gradient = ctx.createLinearGradient(fromParticle.x, fromParticle.y, toParticle.x, toParticle.y);
    gradient.addColorStop(0, `rgba(147, 197, 253, ${connection.opacity})`);
    gradient.addColorStop(0.5, `rgba(147, 197, 253, ${connection.opacity * 0.8})`);
    gradient.addColorStop(1, `rgba(147, 197, 253, ${connection.opacity})`);

    ctx.strokeStyle = gradient;
    ctx.lineWidth = Math.max(0.5, MAX_LINE_WIDTH - connection.distance / 80);
    ctx.lineCap = 'round';

    ctx.shadowColor = 'rgba(147, 197, 253, 0.5)';
    ctx.shadowBlur = 3;

    ctx.beginPath();
    ctx.moveTo(fromParticle.x, fromParticle.y);
    ctx.lineTo(toParticle.x, toParticle.y);
    ctx.stroke();

    ctx.restore();
  });
};

const render = () => {
  const canvas = canvasRef.value;
  if (!canvas) return;

  const ctx = canvas.getContext('2d');
  if (!ctx) return;

  ctx.clearRect(0, 0, canvasWidth.value, canvasHeight.value);

  drawConnections(ctx);

  drawParticles(ctx);
};

const animate = () => {
  updateParticles();
  updateConnections();
  render();
  animationId.value = requestAnimationFrame(animate);
};

const handleResize = () => {
  updateCanvasSize();
  nextTick(() => {
    initializeParticles();
    connectionMap.value.clear();
    connections.value = [];
  });
};

onMounted(() => {
  updateCanvasSize();
  nextTick(() => {
    initializeParticles();
    animate();
  });
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
