<template>
  <main
    class="min-h-screen bg-[radial-gradient(70%_60%_at_20%_20%,rgba(99,102,241,0.15),transparent_60%),radial-gradient(60%_50%_at_80%_30%,rgba(236,72,153,0.12),transparent_60%),radial-gradient(60%_60%_at_50%_80%,rgba(34,197,94,0.10),transparent_60%),linear-gradient(135deg,#1c1c1c_0%,#111111_100%)] text-white font-sans flex items-center justify-center p-6 md:p-8 relative overflow-hidden"
  >
    <div class="max-w-3xl mx-auto text-center space-y-10 md:space-y-12 relative z-10">
      <div class="space-y-4">
        <h1
          class="text-5xl md:text-7xl font-extrabold tracking-tight bg-[linear-gradient(90deg,#f43f5e_0%,#f59e0b_32%,#a78bfa_68%,#38bdf8_100%)] bg-clip-text text-transparent leading-tight drop-shadow-[0_0_22px_rgba(168,85,247,0.22)]"
        >
          Haii, I'm Averi
        </h1>
        <p class="text-lg md:text-2xl text-slate-300/90 font-normal md:font-light">I write bad code</p>
      </div>

      <div class="space-y-3">
        <div class="grid grid-cols-2 md:grid-cols-4 gap-3 md:gap-4">
          <a
            v-for="(social, index) in socials"
            :key="social.name"
            :href="social.url"
            target="_blank"
            rel="noopener noreferrer"
            class="group flex flex-col items-center p-4 rounded-xl bg-white/5 backdrop-blur-md border border-white/10 hover:bg-white/10 hover:border-white/20 transition-all duration-300 hover:scale-[1.04] hover:shadow-[0_10px_30px_-10px_rgba(255,255,255,0.35)] focus:outline-none focus-visible:ring-2 focus-visible:ring-cyan-400/60 focus-visible:ring-offset-0"
            :style="{ 'animation-delay': `${0.5 + index * 0.1}s` }"
          >
            <component
              :is="social.icon"
              class="w-8 h-8 mb-2 transition-colors duration-300 drop-shadow-sm group-hover:drop-shadow-[0_6px_16px_rgba(34,211,238,0.35)]"
              :class="social.color"
            />
            <span
              class="text-sm font-medium text-slate-300 group-hover:text-white transition-colors duration-300 select-none"
            >
              {{ social.name }}
            </span>
          </a>
        </div>
      </div>
    </div>
  </main>
</template>

<script setup lang="ts">``
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
</script>
