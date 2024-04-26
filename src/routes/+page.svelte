<script>
    import { onMount} from "svelte";
    import * as d3 from "d3"
    import { scaleLinear } from 'd3-scale';
    import Radio from './Radio.svelte'; // time radio
    import RangeSlider from "svelte-range-slider-pips"; // time slider

    let data = {};
    onMount(
        async () => {
            //data = await d3.csv('https://raw.githubusercontent.com/tututwo/svelte-tutorial-heatmap/main/src/static/data.csv')
            data["overall_year"] = await d3.csv('http://localhost:5173/overall_year.csv')
            data["overall_year_proptype"] = await d3.csv('http://localhost:5173/overall_year_proptype.csv')
            data["city_year"] = await d3.csv('http://localhost:5173/city_year.csv')
            data["city_year_proptype"] = await d3.csv('http://localhost:5173/city_year_proptype.csv')
        }
    )



    let municipalitySelected = 'All Municipalities';
    const MUNICIPALITIES = ['All Municipalities', 'Acton', 'Arlington', 'Ashland', 'Bedford', 'Bellingham', 'Belmont', 'Beverly', 'Bolton', 'Boston', 'Boxborough', 'Braintree', 'Brookline', 'Burlington', 'Cambridge', 'Canton', 'Carlisle', 'Chelsea', 'Cohasset', 'Concord', 'Danvers', 'Dedham', 'Dover', 'Duxbury', 'Essex', 'Everett', 'Foxborough', 'Framingham', 'Franklin', 'Gloucester', 'Hamilton', 'Hanover', 'Hingham', 'Holbrook', 'Holliston', 'Hopkinton', 'Hudson', 'Hull', 'Ipswich', 'Lexington', 'Lincoln', 'Littleton', 'Lynn', 'Lynnfield', 'Malden', 'Manchester', 'Marblehead', 'Marlborough', 'Marshfield', 'Maynard', 'Medfield', 'Medford', 'Medway', 'Melrose', 'Middleton', 'Milford', 'Millis', 'Milton', 'Nahant', 'Natick', 'Needham', 'Newton', 'Norfolk', 'North Reading', 'Norwell', 'Norwood', 'Peabody', 'Pembroke', 'Quincy', 'Randolph', 'Reading', 'Revere', 'Rockland', 'Rockport', 'Salem', 'Saugus', 'Scituate', 'Sharon', 'Sherborn', 'Somerville', 'Southborough', 'Stoneham', 'Stoughton', 'Stow', 'Sudbury', 'Swampscott', 'Topsfield', 'Wakefield', 'Walpole', 'Waltham', 'Watertown', 'Wayland', 'Wellesley', 'Wenham', 'Weston', 'Westwood', 'Weymouth', 'Wilmington', 'Winchester', 'Winthrop', 'Woburn', 'Wrentham'];


    let timeSelected;
	const time_options = [{
		value: '2000-2009',
		label: '2000-2009',
	}, {
		value: '2010-2019',
		label: '2010-2019',
	}, ]

    const YEAR_MIN = 2000;
    const YEAR_MAX = 2022;
    let timeRangeSelected = [YEAR_MIN, YEAR_MAX];

</script>



<h1>Welcome to SvelteKit</h1>
<p>Visit <a href="https://kit.svelte.dev">kit.svelte.dev</a> to read the documentation</p>


<select bind:value={municipalitySelected}>
	{#each MUNICIPALITIES as option}
		<option value={option}>{option}</option>
	{/each}
</select>


<Radio {time_options} fontSize={16} legend='Select a time period' bind:userSelected={timeSelected}/>



<!--

<RangeSlider min={parseInt(YEAR_MIN)} max={parseInt(YEAR_MAX)}/>
<RangeSlider values={[50]} pips/>
<RangeSlider values={[2000, 2022]} min=2000 max=2022 pips range />

<RangeSlider values={[2000, 2022]} min={2000} max={2022}/>
<RangeSlider values={[50, 75]} pips />
-->

<div>

    <RangeSlider bind:values={timeRangeSelected} pips first='label' last='label' all='label'   min={2000} max={2022} range />


</div>


<p>
    Selected Municipality is {municipalitySelected}. <br><br>
    Selected Radio Time is {timeSelected}. <br>
    Selected Range Time is {timeRangeSelected}. <br>
</p>



<style>


</style>