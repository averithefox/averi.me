<template>
  <main class="w-full h-full bg-black text-green-700 font-mono flex items-center justify-center select-none">
    <div class="text-center">
      <h1 class="text-4xl font-bold">Hello, I'm Averi!</h1>
      <p class="text-lg mt-2">I write bad code.</p>
      <div class="mt-4">
        <h2 class="text-2xl font-bold">Socials</h2>
        <ul class="mt-2">
          <li v-for="social in socials" :key="social.name">
            <div>
              <a :href="social.url" target="_blank" class="inline-flex items-center justify-center space-x-2 hover:underline">
                <component class="w-6 h-6" v-bind:is="social.icon" />
                <span>{{ social.name }}</span>
              </a>
            </div>
          </li>
        </ul>
      </div>
    </div>
  </main>
</template>

<script setup lang="ts">
import type { IconType } from "vue-icons-plus";
import { BsDiscord, BsGithub, BsTwitter } from "vue-icons-plus/bs";
import { SiBluesky } from "vue-icons-plus/si";

const socials = [
  {
    name: "GitHub",
    url: "https://github.com/averithefox",
    icon: BsGithub,
  },
  {
    name: "Discord",
    url: "https://discord.com/users/719890634294427669",
    icon: BsDiscord,
  },
  {
    name: "Twitter",
    url: "https://x.com/averithefox",
    icon: BsTwitter,
  },
  {
    name: "Bluesky",
    url: "https://bsky.app/profile/averi.me",
    icon: SiBluesky,
  },
] satisfies { name: string; url: string; icon: IconType }[];
</script>
