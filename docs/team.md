---
layout: page
title: Познакомьтесь с командой
description: Разработка Vite ведется международной командой.
---

<script setup>
import {
  VPTeamPage,
  VPTeamPageTitle,
  VPTeamPageSection,
  VPTeamMembers
} from 'vitepress/theme'
import { core, emeriti } from './_data/team'
</script>

<VPTeamPage>
  <VPTeamPageTitle>
    <template #title>Познакомьтесь с командой</template>
    <template #lead>
      Разработка Vite ведется международной командой, некоторые члены которой
      выбрали быть представленными ниже.
    </template>
  </VPTeamPageTitle>
  <VPTeamMembers :members="core" />
  <VPTeamPageSection>
    <template #title>Почетные члены команды</template>
    <template #lead>
      Здесь мы чтим некоторых уже неактивных членов команды, которые внесли ценный
      вклад в прошлом.
    </template>
    <template #members>
      <VPTeamMembers size="small" :members="emeriti" />
    </template>
  </VPTeamPageSection>
</VPTeamPage>
