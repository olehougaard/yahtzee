<script setup lang="ts">
  import * as api from '../model/api'
  import { die_values } from 'domain/src/model/dice';
  import { computed } from 'vue';
  import type { IndexedYahtzee } from '@/model/game'
  import { slots, type SlotKey, lower_section_keys } from 'domain/src/model/yahtzee.slots';

  const { game, player, enabled } = defineProps<{game: IndexedYahtzee, player: string, enabled: boolean}>()

  const players = computed(() => game.players())
  
  const register = (key: SlotKey) => {
    if (enabled)
      api.register(game, key, player)
  }

  const isActive = (p: string) => {
    return game.playerInTurn() === player && player === p
  }

  const playerScores = (key: SlotKey): { player: string; score: number | undefined }[] => {
    return players.value.map((player, i) => ({player, score: game.score(i).score(key)}))
  }

  const potentialScore = (key: SlotKey) => slots[key].score(game.roll())

  const displayScore = (score: number | undefined): string => {
    if (score === 0)
      return '---'
    else
      return score?.toString() ?? ''
  }

  const activeClass = (p: string) => p === player? 'activeplayer' : undefined
</script>

<template>
    <div class="score">
      <table class="scorecard">
        <tbody>
          <tr class="section_header"><td :colspan='players.length + 2'>Upper Section</td></tr>
          <tr><td>Type</td><td>Target</td><td v-for="player in players" :class="activeClass(player)">{{player}}</td></tr>
          <tr v-for="val in die_values">
            <td>{{val}}s</td>
            <td>{{3 * val}}</td>
            <template v-for="{player, score} in playerScores(val)">
              <td v-if="isActive(player) && score === undefined" class="clickable potential" @click="register(val)">{{displayScore(potentialScore(val))}}</td>
              <td v-else-if="isActive(player)" class="activeplayer">{{displayScore(score)}}</td>
              <td v-else>{{displayScore(score)}}</td>
            </template>
          </tr>
          <tr>
            <td>Sum</td>
            <td>63</td>
            <td v-for="(player, index) in players" :class="activeClass(player)">{{game.score(index).sum()}}</td>
          </tr>
          <tr>
            <td>Bonus</td>
            <td>50</td>
            <td v-for="(player, index) in players" :class="activeClass(player)">{{game.score(index).bonus()}}</td>
          </tr>
          <tr>
            <td>Total</td>
            <td></td>
            <td v-for="(player, index) in players" :class="activeClass(player)">{{game.score(index).total_upper()}}</td>
          </tr>
          <tr class="section_header"><td :colspan='players.length + 2'>Lower Section</td></tr>
          <tr v-for="key in lower_section_keys">
            <td>{{key.charAt(0).toUpperCase() + key.slice(1)}}</td>
            <td></td>
            <template v-for="{player, score} in playerScores(key)">
              <td v-if="isActive(player) && score === undefined" class="clickable potential" @click="register(key)">{{displayScore(potentialScore(key))}}</td>
              <td v-else-if="isActive(player)" class="activeplayer" @click="register(key)">{{displayScore(score)}}</td>
              <td v-else>{{displayScore(score)}}</td>
            </template>
          </tr>
          <tr>
            <td>Total</td>
            <td></td>
            <td v-for="(player, index) in players" :class="activeClass(player)">{{game.score(index).total()}}</td>
          </tr>
        </tbody>
      </table>
    </div>  
</template>

<style>
  .score {
    display: inline-block;
  }
  .scorecard {
    border: 1px solid black;
    border-collapse: collapse;
  }
  .scorecard td {
    border: 1px solid black;
    padding: .3rem;
  }
  .activeplayer {
    font-weight: bold;
  }
  .clickable {
    cursor: pointer;
  }
  .potential {
    color: transparent;
  }
  .potential:hover {
    color: black;
  }
</style>