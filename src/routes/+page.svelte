<script>
    import * as d3 from "d3"
    import { scaleLinear } from 'd3-scale';
    import Radio from './Radio.svelte'; // time radio
    import RangeSlider from "svelte-range-slider-pips"; // time slider
    import * as overall_year_json from './data/overall_year.json';
    import * as year_proptype_json from './data/year_proptype.json';
    import * as city_year_json from './data/city_year.json';
    import * as city_year_proptype_json from './data/city_year_proptype.json';

    let data = {
        "overall_year": overall_year_json,
        "overall_year_proptype": year_proptype_json,
        "city_year": city_year_json,
        "city_year_proptype":city_year_proptype_json,
    };

    function makeDataKey(year, city=undefined, proptype=undefined) {
        year = year.toString();
        if (city === undefined && proptype === undefined) {
            return year
        } else if (city === undefined && proptype !== undefined) {
            //"(2001, 'RCD')"
            return "(".concat(year, ", '", proptype, "')");
        } else if (city !== undefined && proptype !== undefined) {
            //"('Acton', 2000, 'LND')"
            return "('".concat(city, "', ", year, ", '", proptype, "')");
        } else if (city !== undefined && proptype === undefined) {
            //"('Arlington', 2008)"
            return "('".concat(city, "', ", year, ")");
        }
    }

    function pInvestorBuy(dtable, year_min, year_max, muni=undefined, proptype=undefined) {
        console.log("Start calc");
        let num_txn = 0;
        let num_investor_txn = 0;
        for (let y = year_min; y <= year_max; y++) {
            let key = makeDataKey(y, muni, proptype);
            num_txn += dtable.num_txn[key];
            num_investor_txn += dtable.num_investor_txn[key];
        }
        console.log(year_min, year_max, num_txn, num_investor_txn, muni, proptype);
        return num_txn > 0 ? 100*num_investor_txn/num_txn : 0;
    }


    let municipalitySelected = 'Boston';
    const MUNICIPALITIES = ['Acton', 'Arlington', 'Ashland', 'Bedford', 'Bellingham', 'Belmont', 'Beverly', 'Bolton', 'Boston', 'Boxborough', 'Braintree', 'Brookline', 'Burlington', 'Cambridge', 'Canton', 'Carlisle', 'Chelsea', 'Cohasset', 'Concord', 'Danvers', 'Dedham', 'Dover', 'Duxbury', 'Essex', 'Everett', 'Foxborough', 'Framingham', 'Franklin', 'Gloucester', 'Hamilton', 'Hanover', 'Hingham', 'Holbrook', 'Holliston', 'Hopkinton', 'Hudson', 'Hull', 'Ipswich', 'Lexington', 'Lincoln', 'Littleton', 'Lynn', 'Lynnfield', 'Malden', 'Manchester', 'Marblehead', 'Marlborough', 'Marshfield', 'Maynard', 'Medfield', 'Medford', 'Medway', 'Melrose', 'Middleton', 'Milford', 'Millis', 'Milton', 'Nahant', 'Natick', 'Needham', 'Newton', 'Norfolk', 'North Reading', 'Norwell', 'Norwood', 'Peabody', 'Pembroke', 'Quincy', 'Randolph', 'Reading', 'Revere', 'Rockland', 'Rockport', 'Salem', 'Saugus', 'Scituate', 'Sharon', 'Sherborn', 'Somerville', 'Southborough', 'Stoneham', 'Stoughton', 'Stow', 'Sudbury', 'Swampscott', 'Topsfield', 'Wakefield', 'Walpole', 'Waltham', 'Watertown', 'Wayland', 'Wellesley', 'Wenham', 'Weston', 'Westwood', 'Weymouth', 'Wilmington', 'Winchester', 'Winthrop', 'Woburn', 'Wrentham'];

    const PROPTYPES = ['CND', 'R1F', 'R2F', 'R3F', 'LND']

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

    let x_vals = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
    let x_labels = [];
    let y_vals = [];
    $: {
        y_vals = [];
        //PROPTYPES.forEach();
    }

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
    Selected Range Time is {timeRangeSelected}. <br><br>
</p>



{#key timeRangeSelected}

<p>
    Overall % investor buyers {pInvestorBuy(data.overall_year, timeRangeSelected[0], timeRangeSelected[1])}. <br>
    Selected Muni % investor buyers {pInvestorBuy(data.city_year, timeRangeSelected[0], timeRangeSelected[1], municipalitySelected)}. <br>
</p>

{/key}



<style>


</style>