<script>
    import BarsWithBg from './charts/BarsWithBg.svelte'
    import Legend from './common/Legend.svelte'
    import locale from '@reuters-graphics/d3-locale';
    import {dateDiff, approxDate, sNumber} from '../dateDiff'
    import ProgressBar from "@okrad/svelte-progressbar";

    export let data;
    export let height;
    export let index;
    let series = [12,64]

    let width;
    let margin = {bottom:20, top:20, left:4, right:4};
    const loc = new locale('es');

    const f = {
		x: loc.formatTime('%b %e'),
        y: loc.format(',.1d'),
        pct: loc.format(',.1f')
    }
    
    $:diff = dateDiff(new Date('2021-05-16'), data.latest.dateComplete);

    $:sentenceTarget = `${data.latest.ccaa} vacunará a <strong>${loc.format(`,.2r`)(data.latest.shareTarget)}</strong> personas en esta primera fase, según el reparto de vacunas actual.`;
    $:sentenceDiff = (diff <= -7)
        ? `Al ritmo de vacunación actual, ${data.latest.ccaa} completará la primera fase <strong>${approxDate(data.latest.dateComplete)}</strong>, <strong>antes de la previsión de mediados de marzo</strong> del Ministerio de Sanidad.`
        : ( diff <= 7 && diff > -7)
        ? 'Si continúa con el mismo ritmo de vacunación, completará la primera fase <strong>dentro del plazo previsto</strong> por el Ministerio de Sanidad de <strong>mediados de marzo</strong>.'
        : ( diff > 7 && diff <= 21)
        ? `No completará la primera fase hasta <strong>${approxDate(data.latest.dateComplete)}</strong>; <strong>${sNumber(Math.floor(diff / 7), 'f')} ${(Math.floor(diff / 7) === 1) ? 'semana' : 'semanas'} más tarde</strong> de lo previsto por el Ministerio de Sanidad.`
        : `Acumula <strong>${sNumber(Math.floor(diff / 7), 'f')} semanas de retraso</strong> respecto a la previsión del Ministerio de Sanidad y no completará esta primera fase hasta <strong>${approxDate(data.latest.dateComplete)}</strong>.`
    $:sentence = `${sentenceTarget} ${sentenceDiff}`;

    $:tardy = ( diff <= 7)
        ? 'ontime'
        : ( diff > 7 && diff <= 21)
        ? 'late'
        : 'verylate';

    const legendItems = [
        {  
            color:'url(#diagonalHatch)',
            label:'Vacunas entregadas<br/>(cada semana)'
        },
        {  
            class:'blue',
            label:'<strong>Dosis administradas</strong><br/>(diario)'
        }
    ];


</script>

<li class='ccaa'>
    <div class='numbers'>
        <h2 class='anchor' id={data.latest.ccaa}>{data.latest.ccaa}</h2>
        <p class='number'>{f.y(data.latest.entregadas)}</p>
        <p class='number'><strong>{f.y(data.latest.administradas)}</strong></p>
        <p class='number'><strong>{f.pct(data.latest.admin_entregadas)}%</strong></p>
        <p class='number'>{f.y(data.latest.vacuna_completa)}</p>
    </div>
    <p class="date">Última vacuna registrada a {loc.formatTime('%e de %B')(data.latest.hasta)}</p>

    {#if index===0}
    <Legend {legendItems} />
    {/if}

    <div class='chart' style='height:{height + margin.top + margin.bottom}' bind:clientWidth={width}> 
        <BarsWithBg {data} {width} height={height + margin.top + margin.bottom} key={{x: 'hasta', y: 'administradas', bg: 'entregadas' }} format={f} {margin} />
    </div>
    <div style="position:absolute">
        <!-- <button on:click={() => series = [50, 50]}>fill</button>
        <button on:click={() => series = [0, 0]}>clear</button> -->
        <ProgressBar
            style="radial"
            series={[
                { perc: f.y(+data.latest.vacuna_completa/+data.latest.entregadas*100), color: '#4D87FF' },
                { perc: f.y(+data.latest.pfizer/+data.latest.entregadas*100), color: '#9a82fd' },
                { perc: f.y(+data.latest.administradas/+data.latest.entregadas*100), color: '#60fc7f' }
                ]}
            textSize={80}
            stackSeries={false}
            width="30%"
            height="250"
            thickness={10} 
            margin={4 }  
  
        > 
      </ProgressBar>
    </div>
    <div class='estimates'>
        <p class='indent text'>{@html sentence}</p>
        <img class="icon" src="img/{tardy}.svg" role="img" aria-roledescription={approxDate(data.latest.dateComplete)} aria-label="Icono de un temporizador mostrando el retraso en la administración de vacunas en ${data.latest.ccaa}" alt="Icono de un temporizador mostrando el retraso en la administración de vacunas en ${data.latest.ccaa}" />
    </div>
    {#if data.latest.admin_entregadas > 100}
        <p class='indent text'>Hasta el 27 de enero, cada vial entregado computaba como cinco dosis. A partir de esa fecha Pfizer cambia la ficha técnica para contar seis dosis por vial. ¿Por qué? Con <a href='https://www.europarl.europa.eu/doceo/document/P-9-2021-000394_ES.html' target='_blank' rel="noopener">jeringuillas especiales</a> (de volumen muerto bajo) se pueden extraer esas seis dosis y <a href='https://www.ema.europa.eu/en/news/extra-dose-vials-comirnaty-covid-19-vaccine' target='_blank' rel="noopener">las autoridades europeas lo permiten</a>.</p>
    {/if}

</li>

<style>
    .ccaa {
        padding-bottom: 2rem;
    }
    h2, h3, .number {
        margin:0;
        padding:0;
        hyphens: auto;
    }
    h2, .number { font-size: 1.15rem;}
    h3 { 
        font-size: 1rem;
        color: #505050;
    }
    h2, h3 { font-weight: 600;}
    .icon { 
        width: 80%;
        margin: 0 auto;
    }
    .date {
        color:#505050;
        font-size: .9rem;
        text-align: right;
        margin-top:.5rem;
        font-weight: 400;
    }
    .numbers {
        padding:.5rem 0;
        border-top: 1px solid #dcdcdc;
        border-bottom: 1px dashed #dcdcdc;
        position:sticky;
        top:5.7rem;
        background-color: #f2f2f2;
        z-index:10;
    }
    .number {
        text-align: right;
        font-variant-numeric: tabular-nums;
    }
    .estimates {
        display: grid;
        gap: 1rem;
        grid-template-columns: 80% auto;
    }
    .indent {
        margin-left: 0;
    }
    .chart {
        width:100%;
    }
    a {
        color:#333;
        text-decoration: none;
        border-bottom: 1px dashed #333;
        transition: all .3s;
    }
    .anchor, .anchor:hover {
        color:inherit;
        border: none;
        background-color:inherit;
        scroll-margin-top: 6rem;
    }
    a:hover {
        color:#505050;
        background-color:#FFF;
        text-decoration: none;
        border-bottom: 1px solid #333;
    }
    @media screen and (min-width: 640px) {
		.indent {
            margin-left: 8rem;
        }
        .icon { 
            width: 40%;
            margin: 0 auto;
        }
	}
</style>