# Perú TopoJSON 2024

This repo contains scripts and input files to generate a TopoJSON file covering the following geography layers. (You can preview and download individual geographies with [this tool](https://ccalobeto.github.io/latam-cartography/)).


## Running the scripts
First, you will need to install the NodeJS dependencies. Assuming you have NodeJS installed, simply run:

```bash
npm install
```

Next, you will need to run the scripts. This can be done with a single command:

```bash
npm run make-topo
```

You can preview the output locally by running the included Svelte app:

```bash
npm run dev
```

## Editing/updating the config

To update geographies for year-on-year mergers of local authorities or terminations of counties, you should only need to edit the **/input/changes.csv** file.

If you want to make more complex changes (eg. mergers or terminations of other geographies, or additons of new geography types), you will likely also need to edit **/input/ltla14_lookup.csv**, **/scripts/make-lookup.js**, and possibly other files.

## Visualization
This work follows these steps: 
- Division into svelte components.
- Refactoring svelte components to match with Perú area code.
- Changing raster style to carto. 
- Changing scripts to merge cartography of other countries.

## Inspiration
Inspirated in [uk-topojson](https://github.com/ONSvisual/uk-topojson) repo from **ONSvisual**.

