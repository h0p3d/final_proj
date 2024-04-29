<script>
    import * as d3 from "d3"
    import { scaleLinear } from 'd3-scale';
    import Radio from './Radio.svelte'; // buy/sell/flip radio
    import Radio2 from './Radio2.svelte'; // Municipality/Investor radio
    import RangeSlider from "svelte-range-slider-pips"; // time slider
    import * as year_proptype_json from './data/year_proptype.json';
    import * as investor_year_proptype from './data/investor_year_proptype.json';
    import * as city_year_proptype_json from './data/city_year_proptype.json';
    import { tooltip } from './tooltip';

    let data = {
        "overall_year_proptype": year_proptype_json.default,
        "city_year_proptype":city_year_proptype_json.default,
        'investor_data': investor_year_proptype.default,
    };

    function displayInvestor(inv) {

    }

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

    function calcCityPercent(dtable, year_min, year_max, muni=undefined, proptype=undefined) {
        //console.log("Start calc");
        let num_txn = 0;
        let num_investor_txn = 0;
        let num_investor_sell = 0
        let num_flip = 0;
        let num_investor_flip = 0;
        for (let y = year_min; y <= year_max; y++) {
            let key = makeDataKey(y, muni, proptype);
            num_txn += dtable.num_txn[key] || 0;
            num_flip += dtable.num_flip[key] || 0;
            num_investor_flip += dtable.num_investor_flip[key] || 0;
            num_investor_txn += dtable.num_investor_txn[key] || 0;
            num_investor_sell += dtable.num_investor_sell[key] || 0;
        }
        //console.log(year_min, year_max, num_txn, num_investor_txn, muni, proptype);
        let buy_val = num_txn > 0 ? 100*num_investor_txn/num_txn : 0;
        let sell_val = num_txn > 0 ? 100*num_investor_sell/num_txn : 0;
        let flip_val = num_flip > 0 ? 100*num_investor_flip/num_flip : 0;
        let buy_desc = "".concat(
        "% Investor Buys:        ", Math.round(buy_val, 1),"%",
        "<br>", num_investor_txn, " / ", num_txn,
        " (Investor Buys / Total Buys)",
        );
        let sell_desc = "".concat(
        "% Investor Sells:        ", Math.round(sell_val, 1),"%",
        "<br>", num_investor_sell, " / ", num_txn,
        " (Investor Sells / Total Sales)",
        );
        let flip_desc = "".concat(
        "% Investor Flips:        ", Math.round(flip_val, 1),"%",
        "<br>", num_investor_flip, " / ", num_flip,
        " (Investor flips / Total flips)",
        );
        let desc='<br>';
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
        return {"buy_val": buy_val, "buy_desc": buy_desc.concat(desc),
                "sell_val": sell_val, "sell_desc": sell_desc.concat(desc),
                "flip_val": flip_val, "flip_desc": flip_desc.concat(desc),
        };
    }


    function calcInvestorPercent(inv, year_min, year_max) {
        let invData = {};
        let all_investor_buy = 0;
        let all_investor_sell = 0;
        let all_investor_flip = 0;

        for (const proptype in PROPTYPES) {

            let num_investor_buy = 0;
            let num_investor_sell = 0;
            let num_investor_flip = 0;

            let dtable = data.investor_data[inv];
            console.log("Start calc", inv, proptype, dtable);
            for (let y = year_min; y <= year_max; y++) {
                if (dtable !== undefined && dtable[y] !== undefined && dtable[y][proptype] !== undefined){
                    num_investor_flip += dtable[y][proptype].flip;
                    num_investor_buy += dtable[y][proptype].buy;
                    num_investor_sell += dtable[y][proptype].sell;

                    all_investor_buy += dtable[y][proptype].buy;
                    all_investor_sell += dtable[y][proptype].sell;
                    all_investor_flip += dtable[y][proptype].flip;
                }
            }
            invData[proptype] = {"buy_val": num_investor_buy, "buy_desc": '',
                    "sell_val": num_investor_sell, "sell_desc": '',
                    "flip_val": num_investor_flip, "flip_desc": '',
            };

        }

        for (const proptype in PROPTYPES) {
            //console.log(year_min, year_max, num_txn, num_investor_txn, muni, proptype);
            let result = invData[proptype];
            let buy_val = all_investor_buy > 0 ? 100*result.buy_val/all_investor_buy : 0;
            let sell_val = all_investor_sell > 0 ? 100*result.sell_val/all_investor_sell : 0;
            let flip_val = all_investor_flip > 0 ? 100*result.flip_val/all_investor_flip : 0;
            let buy_desc = "".concat(
            "% Investor Buys:        ", Math.round(buy_val, 1),"%",
            "<br>", result.buy_val, " / ", all_investor_buy,
            " (", inv, " Buy proptype / ", inv, " Buys)",
            );
            let sell_desc = "".concat(
            "% Investor Sells:        ", Math.round(sell_val, 1),"%",
            "<br>", result.sell_val, " / ", all_investor_sell,
            " (", inv, " Sell proptype / ", inv, " Sales)",
            );
            let flip_desc = "".concat(
            "% Investor Flips:        ", Math.round(flip_val, 1),"%",
            "<br>", result.flip_val, " / ", all_investor_flip,
            " (", inv, " Flip proptype / ", inv, " Flips)",
            );
            let desc='<br>';
            if (proptype !== undefined) {
                desc = desc.concat("<br>Type: ", PROPTYPES[proptype]);
            } else {
                desc= desc.concat("<br>Type: ", "All Residential Properties");
            }
            result["buy_val"] = buy_val;
            result["sell_val"] = sell_val;
            result["flip_val"] = flip_val;
            result["buy_desc"] = buy_desc.concat(desc);
            result["sell_desc"] = sell_desc.concat(desc);
            result["flip_desc"] = flip_desc.concat(desc);
        }

        return invData;
    }


    const TOP25INVESTORS = ['FEDERAL NATIONAL MORTGAGE ASSOCIATION (FANNIE MAE)', 'DEUTSCHE BANK NATIONAL TRUST COMPANY', 'US BANK', 'WELLS FARGO BANK', 'THE BANK OF NEW YORK MELLON', 'FEDERAL HOME LOAN MORTGAGE CORPORATION (FREDDIE MAC)', 'HSBC BANK USA', 'BANK OF AMERICA', 'JPMORGAN CHASE BANK', 'MERS', 'CITIBANK', 'GMAC MORTGAGE CORP', 'NATIONSTAR MORTGAGE LLC', 'LASALLE BANK', 'WILMINGTON SVGS FUND SOC', 'AURORA LOAN SVCS LLC', 'USA HUD', 'PRUDENTIAL RELOCATION INC', 'COUNTRYWIDE HOME LOANS', 'BAC HOME LOANS SVCNG LP', 'SIRVA RELOCATION LLC', 'NSP RESIDENTIAL LLC', 'FANNIE MAE', 'PULTE HOMES OF NE LLC', 'REGATTA RIVERVIEW LLC'];
    let investorSelected = 'MERS';

    let municipalitySelected = 'Boston';
    const MUNICIPALITIES = ['Acton', 'Arlington', 'Ashland', 'Bedford', 'Bellingham', 'Belmont', 'Beverly', 'Bolton', 'Boston', 'Boxborough', 'Braintree', 'Brookline', 'Burlington', 'Cambridge', 'Canton', 'Carlisle', 'Chelsea', 'Cohasset', 'Concord', 'Danvers', 'Dedham', 'Dover', 'Duxbury', 'Essex', 'Everett', 'Foxborough', 'Framingham', 'Franklin', 'Gloucester', 'Hamilton', 'Hanover', 'Hingham', 'Holbrook', 'Holliston', 'Hopkinton', 'Hudson', 'Hull', 'Ipswich', 'Lexington', 'Lincoln', 'Littleton', 'Lynn', 'Lynnfield', 'Malden', 'Manchester', 'Marblehead', 'Marlborough', 'Marshfield', 'Maynard', 'Medfield', 'Medford', 'Medway', 'Melrose', 'Middleton', 'Milford', 'Millis', 'Milton', 'Nahant', 'Natick', 'Needham', 'Newton', 'Norfolk', 'North Reading', 'Norwell', 'Norwood', 'Peabody', 'Pembroke', 'Quincy', 'Randolph', 'Reading', 'Revere', 'Rockland', 'Rockport', 'Salem', 'Saugus', 'Scituate', 'Sharon', 'Sherborn', 'Somerville', 'Southborough', 'Stoneham', 'Stoughton', 'Stow', 'Sudbury', 'Swampscott', 'Topsfield', 'Wakefield', 'Walpole', 'Waltham', 'Watertown', 'Wayland', 'Wellesley', 'Wenham', 'Weston', 'Westwood', 'Weymouth', 'Wilmington', 'Winchester', 'Winthrop', 'Woburn', 'Wrentham'];

    const PROPTYPES = {'RCD': 'Condo',
                       'R1F': 'One Family Home',
                       'R2F': 'Two Family Home',
                       'R3F': 'Three Family Home',
                       'LND': 'Residential Land'};


    // radio buy / sell / flip options
	const graph_type_options = [{
		value: 'buy',
		label: 'Buy',
	}, {
		value: 'sell',
		label: 'Sell',
	}, {
		value: 'flip',
		label: 'Flip',
	},];

    let graphTypeSelected = 'buy';

    const graph_type_keys = {
        "buy": {"title": "Buy", "val": "buy_val", "desc": "buy_desc"},
        "sell": {"title": "Sell", "val": "sell_val", "desc": "sell_desc"},
        "flip": {"title": "Flip", "val": "flip_val", "desc": "flip_desc"},
    };


    // choose investor / municipality options
    const comparison_options = [{
		value: 'city',
		label: 'Municipality',
	}, {
		value: 'inv',
		label: 'Investor',
	},
    ];

    let comparisonSelected = 'city';

    // time range slider options

    const YEAR_MIN = 2000;
    const YEAR_MAX = 2022;
    let timeRangeSelected = [YEAR_MIN, YEAR_MAX];


    // bar chart stuff
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
    let y_vals = [];
    let x_color_labels = [];

    let xScale, colorScale, barWidth;
    $: {
        x_vals = [];
        x_color_labels = [];
        y_vals = [];
        let investor_vals = {};
        if (comparisonSelected !== 'city'){
            investor_vals = calcInvestorPercent(investorSelected, timeRangeSelected[0], timeRangeSelected[1]);
        }
        for (const proptype in PROPTYPES) {
            // specific
            if (comparisonSelected === 'city'){
                y_vals.push(calcCityPercent(data.city_year_proptype, timeRangeSelected[0], timeRangeSelected[1], municipalitySelected, proptype));
                x_vals.push(municipalitySelected.slice(0, 3));
            } else {
                //y_vals.push(calcCityPercent(data.city_year_proptype, timeRangeSelected[0], timeRangeSelected[1], municipalitySelected, proptype));
                //x_vals.push(municipalitySelected.slice(0, 3));
                y_vals.push(investor_vals[proptype]);
                x_vals.push(investorSelected.slice(0, 3));
            }
            // all MAPC region
            y_vals.push(calcCityPercent(data.overall_year_proptype, timeRangeSelected[0], timeRangeSelected[1], undefined, proptype));
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



<Radio2 {comparison_options} fontSize={16} legend='Select a comparison' bind:userComparisonSelected={comparisonSelected}/>


{#if comparisonSelected === 'city'}

Select a municipality:

<select bind:value={municipalitySelected}>
	{#each MUNICIPALITIES as option}
		<option value={option}>{option}</option>
	{/each}
</select>

{:else}

Select an investor:

<select bind:value={investorSelected}>
	{#each TOP25INVESTORS as option}
		<option value={option}>{option}</option>
	{/each}
</select>

{/if}


<Radio {graph_type_options} fontSize={16} legend='Select a graph' bind:userSelected={graphTypeSelected}/>

<h3>Select a time range:</h3>
<RangeSlider bind:values={timeRangeSelected} pips first='label' last='label' all='label'   min={2000} max={2022} range />



<div class="chart" bind:clientWidth={bar_graph_width} bind:clientHeight={bar_graph_height}>
	<svg>
        <!-- title -->
        <g class="tick title">
            <text y={bar_padding.top/3-5} x={bar_padding.left + (bar_graph_width-bar_padding.left-bar_padding.right)/2}>
            {#if comparisonSelected === 'city'}
            All Investors {graph_type_keys[graphTypeSelected].title} % in {municipalitySelected} {" "} {timeRangeSelected[0]}-{timeRangeSelected[1]}
            {:else}
            {investorSelected} {graph_type_keys[graphTypeSelected].title} % {timeRangeSelected[0]}-{timeRangeSelected[1]}
            {/if}
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
			{#each y_vals as point, i}
                    <rect
                        title={point[graph_type_keys[graphTypeSelected].desc]}
                        use:tooltip
                        x={xScale(i)+2}
                        y={yScale(point[graph_type_keys[graphTypeSelected].val])}
                        width={barWidth - 4}
                        height={yScale(0) - yScale(point[graph_type_keys[graphTypeSelected].val])}
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
                <text x={bar_padding.left+(bar_graph_width-bar_padding.left)/2} y={bar_padding.bottom-3}>
                    {comparisonSelected === 'city' ? 'Municipality' : 'Investor'}
                </text>
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
    Selected Municipality is {municipalitySelected}. <br>
    Selected Investor is {investorSelected}. <br>
    Selected Comparison is {comparisonSelected}. <br>
    Selected Graph is {graphTypeSelected}. <br>
    Selected Years is {timeRangeSelected}. <br><br>
</p>





<style>

.panel {
    max-width: 1500px;
}

.chart {
		width: 100%;
        min-width: 500px;
		max-width: 500px;
		margin: 0 auto;
        display: inline-flex;
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