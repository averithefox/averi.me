<template>
  <main
    class="min-h-screen bg-gradient-to-br from-slate-900 via-purple-900 to-slate-900 text-white font-sans flex items-center justify-center p-4 relative overflow-hidden"
  >
    <div class="absolute inset-0 pointer-events-none">
      <div
        v-for="i in 32"
        :key="i"
        class="absolute w-3 h-3 text-nowrap"
        :style="getPetalStyle(i)"
        :class="getPetalClass(i)"
      >
        {{ Math.random() < 0.01 ? 'rawr x3' : '🌸' }}
      </div>
    </div>

    <div class="max-w-2xl mx-auto text-center space-y-8 relative z-10">
      <div class="space-y-4">
        <h1
          class="text-5xl md:text-6xl font-bold bg-gradient-to-r from-blue-400 via-purple-400 to-pink-400 bg-clip-text text-transparent leading-tight"
        >
          Haii, I'm Averi
        </h1>
        <p class="text-xl md:text-2xl text-slate-300 font-light">I write bad code</p>
      </div>

      <div class="space-y-6">
        <h2 class="text-2xl font-semibold text-slate-200">Connect with me</h2>
        <div class="grid grid-cols-2 md:grid-cols-4 gap-4">
          <a
            v-for="social in socials"
            :key="social.name"
            :href="social.url"
            target="_blank"
            rel="noopener noreferrer"
            class="group flex flex-col items-center p-4 rounded-xl bg-white/5 backdrop-blur-sm border border-white/10 hover:bg-white/10 hover:border-white/20 transition-all duration-300 hover:scale-105 hover:shadow-lg"
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

const getPetalStyle = (index: number) => {
  const delay = (index * 0.5) % 16;
  const startHeight = index <= 10 ? -10 - index * 5 : -80 - Math.random() * 40;
  const duration = 14 + Math.random() * 4;

  return {
    left: Math.random() * 100 + '%',
    '--start-y': `${startHeight}vh`,
    animationDelay: delay + 's',
    animationDuration: duration + 's',
    opacity: 0
  };
};

const getPetalClass = (index: number) => {
  const animations = ['animate-float', 'animate-float-sway', 'animate-float-drift'];
  return animations[index % 3];
};
</script>

<style scoped>
@keyframes float {
  0% {
    transform: translateY(var(--start-y, -100vh)) rotate(0deg);
    opacity: 0;
  }
  5% {
    opacity: 0.6;
  }
  95% {
    opacity: 0.6;
  }
  100% {
    transform: translateY(100vh) rotate(360deg);
    opacity: 0;
  }
}

.animate-float {
  animation: float linear infinite;
}

@keyframes float-sway {
  0% {
    transform: translateY(var(--start-y, -100vh)) translateX(0) rotate(0deg);
    opacity: 0;
  }
  5% {
    opacity: 0.4;
  }
  25% {
    transform: translateY(calc(var(--start-y, -100vh) + 50vh)) translateX(20px) rotate(90deg);
  }
  50% {
    transform: translateY(calc(var(--start-y, -100vh) + 100vh)) translateX(-10px) rotate(180deg);
  }
  75% {
    transform: translateY(calc(var(--start-y, -100vh) + 150vh)) translateX(15px) rotate(270deg);
  }
  95% {
    opacity: 0.4;
  }
  100% {
    transform: translateY(calc(var(--start-y, -100vh) + 200vh)) translateX(0) rotate(360deg);
    opacity: 0;
  }
}

.animate-float-sway {
  animation: float-sway linear infinite;
}

@keyframes float-drift {
  0% {
    transform: translateY(var(--start-y, -100vh)) translateX(0) rotate(0deg) scale(0.8);
    opacity: 0;
  }
  5% {
    opacity: 0.3;
  }
  30% {
    transform: translateY(calc(var(--start-y, -100vh) + 60vh)) translateX(-25px) rotate(120deg) scale(1.1);
  }
  60% {
    transform: translateY(calc(var(--start-y, -100vh) + 120vh)) translateX(30px) rotate(240deg) scale(0.9);
  }
  95% {
    opacity: 0.3;
  }
  100% {
    transform: translateY(calc(var(--start-y, -100vh) + 200vh)) translateX(-5px) rotate(360deg) scale(0.8);
    opacity: 0;
  }
}

.animate-float-drift {
  animation: float-drift linear infinite;
}
</style>
