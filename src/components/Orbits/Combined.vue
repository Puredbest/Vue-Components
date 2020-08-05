<template>
    <div>
        <iv-orbits style="height: 20vh;" @energies="updatePlot"></iv-orbits>
        <iv-eff-pot  ref="effPlot"></iv-eff-pot>
    </div>
</template>

<script>
export default {
    name: 'iv-orbits-combo',    
    methods:{
        updatePlot(energies){
            console.log('updatePlotParent');
            this.$refs.effPlot.updatePlot(energies);     
        }

    }
}
</script>

<style>

</style>