<template>
  <v-data-table
    :headers="headers"
    :items="results"
    :hide-actions="true"
  >
    <template slot="items" slot-scope="props">
      <td class="text-xs-center">{{ props.item.idx }}</td>
      <td>{{ props.item.scramble }}</td>
      <td class="text-xs-center">{{ props.item.cross.toFixed(2) }}</td>
      <td class="text-xs-center">{{ props.item.f2l.toFixed(2) }}</td>
      <td class="text-xs-center">{{ props.item.oll.toFixed(2) }}</td>
      <td class="text-xs-center">{{ props.item.pll.toFixed(2) }}</td>
      <td class="text-xs-center">{{ props.item.auf.toFixed(2) }}</td>
      <td class="text-xs-center">{{ props.item.raw.percentage.cross.toFixed(2) }} %</td>
      <td class="text-xs-center">{{ props.item.raw.percentage.f2l.toFixed(2) }} %</td>
      <td class="text-xs-center">{{ props.item.raw.percentage.oll.toFixed(2) }} %</td>
      <td class="text-xs-center">{{ props.item.raw.percentage.pll.toFixed(2) }} %</td>
      <td class="text-xs-center">{{ props.item.raw.percentage.auf.toFixed(2) }} %</td>
      <td class="text-xs-center">{{ props.item.turns }}</td>
      <td class="text-xs-center">{{ props.item.time.toFixed(2) }}</td>
      <td class="text-xs-center">{{ props.item.tps }}</td>
    </template>
  </v-data-table>
</template>



<script lang="ts">
import { Component, Vue, Prop } from 'vue-property-decorator'
import { EventHub, Events } from '@/classes/event-hub'
import Timer from '@/classes/timer'
import GameState from '@/classes/game-state'
import { Giiker } from '@/classes/giiker'

@Component
export default class Results extends Vue {
    @Prop()
    private game!: GameState

    private localStorageKey = 'results'

    private headers = [
      { text: 'SN', value: 'sn', align: 'center' },
      {
        text: 'Scramble',
        align: 'left',
        sortable: false,
        value: 'name'
      },
      { text: 'Cross', value: 'cross', align: 'center' },
      { text: 'F2L', value: 'f2l', align: 'center' },
      { text: 'OLL', value: 'oll', align: 'center' },
      { text: 'PLL', value: 'pll', align: 'center' },
      { text: 'AUF', value: 'auf', align: 'center' },
      { text: 'Cross %', value: 'cross', align: 'center' },
      { text: 'F2L %', value: 'f2l', align: 'center' },
      { text: 'OLL %', value: 'oll', align: 'center' },
      { text: 'PLL %', value: 'pll', align: 'center' },
      { text: 'AUF %', value: 'auf', align: 'center' },
      { text: 'Turns', value: 'turns', align: 'center' },
      { text: 'Time', value: 'time', align: 'center', sortable: true },
      { text: 'TPS', value: 'tps', align: 'center' }
    ]

    private results: object[] = []

    private mounted() {
        EventHub.$on(Events.cubeSolved, () => this.onCubeSolved())

        this.loadResults()
    }

    private seconds(milliseconds: number): number {
      return milliseconds / 1000
    }

    private onCubeSolved() {
        this.results.unshift({
            scramble: this.game.scramble.join(' ') || 'Unknown',
            cross: this.seconds(Timer.getCrossSolveTime()),
            f2l: this.seconds(
              Timer.getF2l1SolveTime() +
              Timer.getF2l2SolveTime() +
              Timer.getF2l3SolveTime() +
              Timer.getF2l4SolveTime() +
              Timer.getF2l1InspectionTime() +
              Timer.getF2l2InspectionTime() +
              Timer.getF2l3InspectionTime() +
              Timer.getF2l4InspectionTime()
              // Timer.getF2lInspectionAndSolveTime()
            ),
            oll: this.seconds(Timer.getOllSolveTime() + Timer.getOllInspectionTime()),
            pll: this.seconds(Timer.getPllSolveTime() + Timer.getPllInspectionTime()),
            auf: this.seconds(Timer.getAUFTime()),
            turns: this.game.turns,
            tps: this.game.tps(),
            time: this.game.time / 1000,
            solution: Giiker.getSolution().join(' ') || 'Unknown',
            raw: {
              scramble: this.game.scramble.join(' ') || 'Unknown',
              cross: Timer.getCrossSolveTime(),
              f2l: {
                f2l1: {
                  inspection: Timer.getF2l1InspectionTime(),
                  solve: Timer.getF2l1SolveTime()
                },
                f2l2: {
                  inspection: Timer.getF2l2InspectionTime(),
                  solve: Timer.getF2l2SolveTime()
                },
                f2l3: {
                  inspection: Timer.getF2l3InspectionTime(),
                  solve: Timer.getF2l3SolveTime()
                },
                f2l4: {
                  inspection: Timer.getF2l4InspectionTime(),
                  solve: Timer.getF2l4SolveTime()
                }
              },
              oll: {
                inspection: Timer.getOllInspectionTime(),
                solve: Timer.getOllSolveTime()
              },
              pll: {
                inspection: Timer.getPllInspectionTime(),
                solve: Timer.getPllSolveTime()
              },
              auf: Timer.getAUFTime(),
              percentage: {
                cross: Timer.getCrossSolveTime() / this.game.time * 100,
                f2l: (
                  Timer.getF2l1SolveTime() +
                  Timer.getF2l2SolveTime() +
                  Timer.getF2l3SolveTime() +
                  Timer.getF2l4SolveTime() +
                  Timer.getF2l1InspectionTime() +
                  Timer.getF2l2InspectionTime() +
                  Timer.getF2l3InspectionTime() +
                  Timer.getF2l4InspectionTime()
                  // Timer.getF2lInspectionAndSolveTime()
                ) / this.game.time * 100,
                oll: (Timer.getOllSolveTime() + Timer.getOllInspectionTime()) / this.game.time * 100,
                pll: (Timer.getPllSolveTime() + Timer.getPllInspectionTime()) / this.game.time * 100,
                auf: Timer.getAUFTime() / this.game.time * 100
              }
            }
        })
        // this.results = this.results.filter(r => 'Sum'.localeCompare(r.scramble) != 0)
        localStorage.setItem(this.localStorageKey, JSON.stringify(this.results))

        this.loadResults()
        // window.hihi = Timer
    }

    private loadResults() {
      try {
        const value = localStorage.getItem(this.localStorageKey)

        if (value) {
          this.results = JSON.parse(value)
          this.results = this.results.filter((r: any) => 'Sum'.localeCompare(r.scramble) != 0)
          let tmp = {
            raw: {
              cross: 0,
              f2l: 0,
              oll: 0,
              pll: 0,
              auf: 0,
              turns: 0,
              time: 0,
              total: 0
            },
            percentage: {
              cross: 0,
              f2l: 0,
              oll: 0,
              pll: 0,
              auf: 0
            }
          }
          let idx = this.results.length
          for(let i = 0; i < this.results.length; i++) {
            let solve: any
            solve = this.results[i]
            solve.idx = idx--
            tmp.raw.cross += solve.cross
            tmp.raw.f2l += solve.f2l
            tmp.raw.oll += solve.oll
            tmp.raw.pll += solve.pll
            tmp.raw.auf += solve.auf

            tmp.raw.turns += solve.turns
            tmp.raw.time += solve.time
          }
          tmp.raw.total += tmp.raw.cross
          tmp.raw.total += tmp.raw.f2l
          tmp.raw.total += tmp.raw.oll
          tmp.raw.total += tmp.raw.pll
          tmp.raw.total += tmp.raw.auf

          tmp.percentage.cross = tmp.raw.cross / tmp.raw.total * 100
          tmp.percentage.f2l = tmp.raw.f2l / tmp.raw.total * 100
          tmp.percentage.oll = tmp.raw.oll / tmp.raw.total * 100
          tmp.percentage.pll = tmp.raw.pll / tmp.raw.total * 100
          tmp.percentage.auf = tmp.raw.auf / tmp.raw.total * 100

          this.results.unshift({
            scramble: 'Sum',
            cross: tmp.raw.cross,
            f2l: tmp.raw.f2l,
            oll: tmp.raw.oll,
            pll: tmp.raw.pll,
            auf: tmp.raw.auf,
            raw: {
              percentage: tmp.percentage
            },
            time: tmp.raw.time,
            turns: tmp.raw.turns
          })

          console.log('this.results length', this.results.length);
          console.log(this.results);
        }

      } catch (Exception) {
        this.results = []
      }
    }
}
</script>
