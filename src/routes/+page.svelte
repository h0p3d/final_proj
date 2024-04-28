<script>
    import * as d3 from "d3"
    import { scaleLinear } from 'd3-scale';
    import Radio from './Radio.svelte'; // time radio
    import RangeSlider from "svelte-range-slider-pips"; // time slider
    import * as overall_year_json from './data/overall_year.json';
    import * as year_proptype_json from './data/year_proptype.json';
    import * as city_year_json from './data/city_year.json';
    import * as city_year_proptype_json from './data/city_year_proptype.json';
    import { tooltip } from './tooltip';

    let data = {
        "overall_year": overall_year_json,
        "overall_year_proptype": year_proptype_json,
        "city_year": city_year_json,
        "city_year_proptype":city_year_proptype_json,
    };

    function makeDataKey(year, city=undefined, proptype=undefined) {
        year = year.toString();
        if (city === undefined && proptype === undefined) {
            return year // "2001"
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
        //console.log("Start calc");
        let num_txn = 0;
        let num_investor_txn = 0;
        for (let y = year_min; y <= year_max; y++) {
            let key = makeDataKey(y, muni, proptype);
            num_txn += dtable.num_txn[key] || 0;
            num_investor_txn += dtable.num_investor_txn[key] || 0;
        }
        //console.log(year_min, year_max, num_txn, num_investor_txn, muni, proptype);
        let val = num_txn > 0 ? 100*num_investor_txn/num_txn : 0
        let desc = "".concat(
        "% Investor Buys:        ", Math.round(val, 1),"%",
        "<br>", num_investor_txn, " / ", num_txn,
        " (Investor Buys / Total Buys)",
        );
        if (muni !== undefined) {
            desc= desc.concat("<br>City:&nbsp; ", muni);
        } else {
            desc= desc.concat("<br>City:&nbsp; ", "All Municipalities");
        }
        if (proptype !== undefined) {
            desc = desc.concat("<br>Type: ", PROPTYPES[proptype]);
        } else {
            desc= desc.concat("<br>Type: ", "All Residential Properties");
        }
        return {"val": val, "desc": desc};
    }


    let municipalitySelected = 'Boston';
    const MUNICIPALITIES = ['Acton', 'Arlington', 'Ashland', 'Bedford', 'Bellingham', 'Belmont', 'Beverly', 'Bolton', 'Boston', 'Boxborough', 'Braintree', 'Brookline', 'Burlington', 'Cambridge', 'Canton', 'Carlisle', 'Chelsea', 'Cohasset', 'Concord', 'Danvers', 'Dedham', 'Dover', 'Duxbury', 'Essex', 'Everett', 'Foxborough', 'Framingham', 'Franklin', 'Gloucester', 'Hamilton', 'Hanover', 'Hingham', 'Holbrook', 'Holliston', 'Hopkinton', 'Hudson', 'Hull', 'Ipswich', 'Lexington', 'Lincoln', 'Littleton', 'Lynn', 'Lynnfield', 'Malden', 'Manchester', 'Marblehead', 'Marlborough', 'Marshfield', 'Maynard', 'Medfield', 'Medford', 'Medway', 'Melrose', 'Middleton', 'Milford', 'Millis', 'Milton', 'Nahant', 'Natick', 'Needham', 'Newton', 'Norfolk', 'North Reading', 'Norwell', 'Norwood', 'Peabody', 'Pembroke', 'Quincy', 'Randolph', 'Reading', 'Revere', 'Rockland', 'Rockport', 'Salem', 'Saugus', 'Scituate', 'Sharon', 'Sherborn', 'Somerville', 'Southborough', 'Stoneham', 'Stoughton', 'Stow', 'Sudbury', 'Swampscott', 'Topsfield', 'Wakefield', 'Walpole', 'Waltham', 'Watertown', 'Wayland', 'Wellesley', 'Wenham', 'Weston', 'Westwood', 'Weymouth', 'Wilmington', 'Winchester', 'Winthrop', 'Woburn', 'Wrentham'];

    const PROPTYPES = {'RCD': 'Condo',
                       'R1F': 'One Family Home',
                       'R2F': 'Two Family Home',
                       'R3F': 'Three Family Home',
                       'LND': 'Residential Land'};

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

    const bar_padding = { top: 60, right: 1, bottom: 40, left: 40 };
    let bar_graph_width = 400+bar_padding.left+bar_padding.right;
    let bar_graph_height = 300+bar_padding.top+bar_padding.bottom;
    let yScale = scaleLinear()
		.domain([0, 100])
		.range([bar_graph_height - bar_padding.bottom, bar_padding.top]);
    let y_ticks = [];
    for (let y = 0; y <= 10; y++) {
        y_ticks.push(y*10);
    }
    let x_vals = [];
    let buy_vals = [];
    let x_color_labels = [];

    let xScale, colorScale, barWidth;
    $: {
        x_vals = [];
        x_color_labels = [];
        buy_vals = [];
        for (const proptype in PROPTYPES) {
            buy_vals.push(pInvestorBuy(data.city_year_proptype, timeRangeSelected[0], timeRangeSelected[1], municipalitySelected, proptype));
            buy_vals.push(pInvestorBuy(data.overall_year_proptype, timeRangeSelected[0], timeRangeSelected[1], undefined, proptype));
            x_vals.push(municipalitySelected.slice(0, 3));
            x_vals.push("All");
            x_color_labels.push(proptype);
            x_color_labels.push(proptype);
        };
        colorScale = d3.scaleOrdinal()
        .domain(x_color_labels)
        .range(d3.schemeTableau10);
        xScale = scaleLinear()
		.domain([0, x_vals.length])
		.range([bar_padding.left, bar_graph_width - bar_padding.right]);
        barWidth = barInnerWidth / x_vals.length;
    };

	$: barInnerWidth = bar_graph_width - (bar_padding.left + bar_padding.right);

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



{#key timeRangeSelected}
<div class="chart" bind:clientWidth={bar_graph_width} bind:clientHeight={bar_graph_height}>
	<svg>
        <!-- title -->
        <g class="tick title">
            <text y={bar_padding.top/3-5} x={bar_padding.left + (bar_graph_width-bar_padding.left-bar_padding.right)/2}>
            % Investors in {municipalitySelected} {" "} {timeRangeSelected[0]}-{timeRangeSelected[1]}
            </text>
        </g>

		<!-- y axis -->
		<g class="axis y-axis">
			{#each y_ticks as tick}
				<g class="tick tick-{tick}" transform="translate({bar_padding.left-30}, {yScale(tick)})">
					<line x1="30" x2="100%" />
					<text y="-2" x={tick == 0? "1.5em": "1em" }>{tick === 100? '' : tick}</text>
				</g>
			{/each}
		</g>

		<!-- x axis -->
		<g class="axis x-axis">
			{#each x_vals as point, i}
				<g class="tick" transform="translate({xScale(i)}, {bar_graph_height})">
					<text x={barWidth / 2} y={-bar_padding.bottom/2-4}>{point}</text>
				</g>
			{/each}
		</g>

		<g class="bars">
			{#each buy_vals as point, i}

                    <rect
                        title={point.desc}
                        use:tooltip
                        x={xScale(i)+2}
                        y={yScale(point.val)}
                        width={barWidth - 4}
                        height={yScale(0) - yScale(point.val)}
                        fill= {colorScale(x_color_labels[i])}
                    />

			{/each}
		</g>

        <!-- proptype borders -->
        <g class="axis proptype border">
            {#each x_color_labels as point, i}

            {#if i%2 === 0}
                <g class="tick tick-proptype" transform="translate({xScale(i+2)}, 0)">
                    <text x={-barWidth} y={bar_padding.top-3}>{PROPTYPES[point].replace("Home", "")}</text>
                    <line y1={2*bar_padding.top/3} y2={bar_graph_height-bar_padding.bottom/2} />
                </g>
            {/if}

            {/each}
        </g>

        <!-- axis borders -->
        <g class="axis border">

            <g class="tick tick-border" transform="translate(0, {yScale(0)})">
                <text x={bar_padding.left+(bar_graph_width-bar_padding.left)/2} y={bar_padding.bottom-3}>Municipality</text>
                <line x1="0" x2="100%" />
            </g>
            <g class="tick tick-border" transform="translate(0, {bar_padding.top})">
                <text x={bar_padding.left+(bar_graph_width-bar_padding.left)/2} y={-bar_padding.top/3-3}>Property Type</text>
                <line x1="0" x2="100%" />
            </g>
            <g class="tick tick-border">
                <text x=0 y=0 transform="translate({bar_padding.left/2-5}, {15+bar_graph_height/2}) rotate(270)">Percentage</text>
                <line y1="0%" y2="100%" transform="translate({bar_padding.left-1}, 0)"/>
            </g>
            <g class="tick tick-border" transform="translate({bar_graph_width-1}, 0)">
                <line y1="0%" y2="100%" />
            </g>

        </g>
	</svg>
</div>


<p>
    Selected Municipality is {municipalitySelected}. <br><br>
    Selected Radio Time is {timeSelected}. <br>
    Selected Range Time is {timeRangeSelected}. <br><br>
</p>


<p>
    Overall % investor buyers {pInvestorBuy(data.overall_year, timeRangeSelected[0], timeRangeSelected[1])}. <br>
    Selected Muni % investor buyers {pInvestorBuy(data.city_year, timeRangeSelected[0], timeRangeSelected[1], municipalitySelected)}. <br>
    buy vals are {buy_vals}.<br>
    x vals are {x_vals}.
</p>
{/key}



<style>

.chart {
		width: 100%;
		max-width: 500px;
		margin: 0 auto;
	}

	svg {
		position: relative;
		width: 100%;
		height: 400px;
	}

	.tick {
		font-family: Helvetica, Arial;
		font-size: 9pt;
		font-weight: 200;
	}

	.tick line {
		stroke: #abaaaa;
		stroke-dasharray: 0;
	}

    .tick.tick.tick-border text {
        font-size: 12pt;
		fill: #656565;
		text-anchor: middle;
	}

    .tick.tick-border line {
        stroke: #656565 !important;
        stroke-width: 2;
	}

    .tick.tick-proptype line {
        stroke: #abaaaa !important;
        stroke-width: 1.25;
	}


	.tick.tick.tick-proptype text {
		fill: #abaaaa;
		text-anchor: middle;
	}

    .tick.title text {
        font-weight: bold;
        font-size: 14pt;
		fill: #000000;
		text-anchor: middle;
	}

	.tick text {
		fill: #abaaaa;
		text-anchor: start;
        text-align: right;
	}

	.x-axis .tick text {
		text-anchor: middle;
        text-align: middle;
	}

</style>