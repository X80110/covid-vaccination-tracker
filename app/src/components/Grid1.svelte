<script>
	// export let grid = [2, 6];
	export let selectedCCAA;
	import locale from "@reuters-graphics/d3-locale";
	import GapChartTiny from "./charts/GapChartTiny.svelte";
	import { max } from "d3-array";

	import * as ccaaData2 from "../../public/gap-chart-ca.json"; //dose2_perc_total
	import * as allCCAA from "../../public/gap-chart-demo.json"; //dose2_perc_total
	// import * as data_raw from "../../public/data_xavier.json"; // downloaded from Xavier's branch //dose2_perc_total
	// import * as data_raw from "../../public/data.json"; // dose2_pct_total
	// import * as data_raw from "../../public/data_xavier.json"; // dose2_perc_total
	import * as data_raw from "../../public/dataGrid.json"; // dose2_perc_total

	export let layout;
	let width, height;

	console.log(selectedCCAA)

	/* --------------------  
	   DATA PREPROCESSING 
	-----------------------*/
	// first vaccination date March 31
	let DataLong = data_raw.default.filter(
		(d) => new Date(d.date) > new Date("2021-03-30")
	);

	let uniqueCCAA = [...new Set(DataLong.map((item) => item.ccaa))];

	// Totales is missing in xavi's dataset, so use "Cataluña" as value1 for now
	let dataGlobal = DataLong.filter((d) => d.ccaa === "Total España").map(
		(d) => {
			return {
				date: new Date(d.date.split("T")[0]),
				dateStr: d.date.split("T")[0],
				ccaa: d.ccaa,
				value1: d.value0,
			};
		}
	);

	console.log("dataGlobal--------");
	console.log(dataGlobal);

	function findValueByDate(dateStr) {
		let result = dataGlobal
			.filter((d) => d.dateStr === dateStr)
			.map((d) => d.value1)[0];
		// fill nan with 0 for value1
		if (isNaN(result)) {
			return 0;
		} else {
			return result;
		}
	}
	let data2 = DataLong.map((d) => {
		return {
			date: new Date(d.date.split("T")[0]),
			dateStr: new Date(d.date.split("T")[0]),
			ccaa: d.ccaa,
			// fill nan with 0 for value0
			value0: isNaN(d["value0"]) ? 0 : d["value0"],
			value1: findValueByDate(d.date.split("T")[0]),
		};
	});
	console.log("data2=========");
	console.log(data2);
	/* --------------------  
	   Set up for <div> 
	-----------------------*/

	// let color = ['rgba(92,198,178, 1)', 'rgba(0,0,0, 1)']

	let color = ["#569E4B", "#F0A81C"];
	// let colorDiff = ['rgba(92,198,178, 0.5)', 'rgba(0,0,0, 0.5)']
	const loc = new locale("en");
	const format = {
		x: loc.formatTime("%B"),
		y: loc.format(",.1f"),
	};

	// $: col = `repeat(${grid[1]}, 1fr)`;
	// $: row = `repeat(${grid[0]}, 1fr)`;

	// accent
	// for xavier's dataset
	// var items = [
	// ['Galicia', 'Asturias', 'Cantabria', 'País Vasco', 'Aragón', 'Cataluña'],
	// [0, 'Castilla y León', 'La Rioja', 'Navarra', 'Com. Valenciana', 0],
	// [0, 'Extremadura', 'Madrid', 'Castilla-La Mancha', 'Murcia', 0],
	// ['Canarias',0, 'Andalucía', 0,0,'Baleares'],
	// ];

	// to loosely match naming in data.json. some unicode problems
	// for demo dataset
	var items = [
		[
			"Galicia",
			"Asturias",
			"Cantabria",
			"País Vasco",
			"Aragón",
			"Cataluña",
		],
		[0, "Castilla y Leon", "La Rioja", "Navarra", "C.Valenciana", 0],
		[0, "Extremadura", "Madrid", "Castilla - La Mancha", "Murcia", 0],
		["Canarias", 0, "Andalucía", 0, 0, "Baleares"],
	];

	function getVaxDataByCCAA(ccaaName) {
		let selected = data2.filter((d) => d.ccaa === ccaaName);

		return selected;
		// for other demo dataset
		// let selected = allCCAA.default.filter(d => d.ccaa === ccaaName)
		// return selected.map(d => {return{
		// 	date: d.date,
		// 	dateStr: new Date (d.dateStr),
		// 	ccaa: d.ccaa,
		// 	value0: d.value0,
		// 	value1: d.value1
		// };})
	}

	function getMaxVaxByCCAA(ccaaName) {
		let selected = data2.filter((d) => d.ccaa === ccaaName);
		let maxValue = max(selected.map((d) => d.value0)) * 100;
		return maxValue.toString().concat("%");
	}

	function getMaxVaxByCCAAColor(ccaaName) {
		let national = data2.filter((d) => d.ccaa === "Total España");
		let nationalMaxValue = max(national.map((d) => d.value0));
		let selected = data2.filter((d) => d.ccaa === ccaaName);
		let maxValue = max(selected.map((d) => d.value0));

		return maxValue > nationalMaxValue
			? "ccaa-vax-label larger"
			: "ccaa-vax-label";
	}

	function getItem(i, j) {
		if (items[i][j] === 0) {
			return " ";
		} else {
			return items[i][j];
		}
	}

	function sanitizedCCAA(ccaaName) {
		switch (true) {
			case ccaaName === "Castilla y Leon":
				return "C.León";
			case ccaaName === "Com. Valenciana":
				return "C.Valenciana";
			case ccaaName === "Castilla - La Mancha":
				return "C.Mancha";
			default:
				return ccaaName;
		}
	}

	function coloring(i, j) {
		if (getItem(i, j) != " ") {
			return "colored ".concat(getItem(i, j));
		} else {
			return "colorless";
		}
	}
</script>

	<GapChartTiny
		data={getVaxDataByCCAA(selectedCCAA)}
		title="Title"
		desc="Description"
		key={{ x: "dateStr", y: ["value0", "value1"] }}
		{format}
		{color}
	/>
<style>
	.tiny-chart {
		height: 80px;
		width: 120px;
	}
/* 
	.container {
		display: grid;
		border-radius: 2px;
		grid-gap: 2px;
	} */
/* 
	.container div {
		display: block;
	} */

	/* .colored {
		border: 1px solid lightgray;
		background: #fff;
	}

	.Canarias {
		transform: translate(65px, 20px);
	}

	.Baleares {
		transform: translate(20px, -250px);
	}

	.ccaa-label {
		font-family: "Merriweather Sans", Arial, sans-serif;
		letter-spacing: 0;
		font-size: 14px;
		font-weight: 400;
		margin-left: 10px;
		padding-top: 7px;
		display: inline-block;
	}

	.ccaa-vax-label {
		font-family: "Merriweather Sans", Arial, sans-serif;
		letter-spacing: 0;
		font-size: 14px;
		font-weight: 700;
		margin-left: 5px;
		margin-top: 10px;
		color: #C4860D;
	}

	.larger {
		color: #569e4b;
	}

	.smaller {
		color: #f0a81c;
	} */
</style>
