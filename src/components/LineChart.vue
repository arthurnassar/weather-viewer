<template>
    <canvas id="myChart" height="40" width="100"></canvas>
</template>

<script lang="ts">
import { defineComponent } from 'vue'
import Chart from 'chart.js/auto'

export default defineComponent({
    name: 'LineChart',
    props: {
        maxTemp: Array,
        minTemp: Array,
        labels: Array,
        label1: String,
        label2: String
    },
    mounted() {
        if (this.label2) {
            const ctx = document.getElementById('myChart') as HTMLCanvasElement
            const myChart = new Chart(ctx, {
                type: 'line',
                data: {
                    labels: this.labels as string[],
                    datasets: [
                        {
                            label: this.label1,
                            data: this.maxTemp,
                            backgroundColor: ['rgba(255, 99, 132, 0.5)'],
                            borderColor: ['rgba(255, 99, 132, 0.5)'],
                            borderWidth: 2
                        },
                        {
                            label: this.label2,
                            data: this.minTemp,
                            backgroundColor: ['rgba(88, 56, 247, 0.5)'],
                            borderColor: ['rgba(88, 56, 247, 0.5)'],
                            borderWidth: 2
                        }
                    ]
                },
                options: {
                    scales: {
                        y: {
                            beginAtZero: true
                        }
                    }
                }
            })
            myChart
        } else {
            const ctx = document.getElementById('myChart') as HTMLCanvasElement
            const myChart = new Chart(ctx, {
                type: 'line',
                data: {
                    labels: this.labels as string[],
                    datasets: [
                        {
                            label: this.label1,
                            data: this.maxTemp,
                            backgroundColor: ['rgba(255, 99, 132, 0.5)'],
                            borderColor: ['rgba(255, 99, 132, 0.5)'],
                            borderWidth: 2
                        }
                    ]
                },
                options: {
                    scales: {
                        y: {
                            beginAtZero: true
                        }
                    }
                }
            })
            myChart
        }
    }
})
</script>

<style lang="scss">
@import '@/assets/scss/global_style';
</style>
