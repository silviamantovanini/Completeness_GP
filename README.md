This is a set of codes to perform completeness simulations on the GLEAM and GLEAM-X data. Thomas Franzen originally developed this code.

The basic procedure is described below. 

There are four steps to follow:


1) Generate a list of random RA and Dec positions where the simulated point sources will be injected.

     Command line example:
     sbatch --time=06:00:00 generate_pos.sh nsrc=30000 region=75,195,-40,-13 sep_min=5 output_dir=$GLEAMX/source_pos

     The input parameters are explained at the top of the script.
     My mosaic covers about 4000 deg^2, so I set the number of simulated sources to 35,000 per region.
     The minimum separation between the simulated sources is set to 5 arcmin to avoid introducing an artificial confusion factor.
     The script will create the output directory.

2) Prepare file(s) listing the fluxes at which to measure the completeness.

     Command line example:
     generate_fluxes.sh flux=-2.3,-0.5,0.1 nfiles=5 output_dir=$GLEAMX/fluxes

     The input parameters are explained at the top of the script. 
     The fluxes can be divided into multiple files to speed up the next step. The script only takes a few seconds to run.

3) Inject point sources into the wideband image and run source findings on the simulated sources.

     Command line example:
     sbatch --array=1-5 --time=08:00:00 inject_sources.sh input_map_dir=*/input_images input_sources=*/source_pos/source_pos.txt flux_dir=*/fluxes sigma=4.0 output_dir=*/inject

     The input parameters are explained at the top of the script. 
     There is one realisation per flux level (i.e. 35,000 sources of the same flux are injected into the wideband mosaic). 
     The script will loop over the fluxes listed in the flux file. 
     The array parameter indicates which flux file to use. So, 'array=1-5’ allows you to run the script in parallel on the five flux files generated in the previous step.

     The script expects to find the following maps in the input map directory:
     - JDGP_wideband.fits
     - JDGP_wideband_rms.fits
     - JDGP_wideband_bkg.fits
     - JDGP_wideband_projpsf_psf.fits
     Change this inside the script if needed.

     Important: the wide-band image should be rescaled to account for ionospheric smearing; otherwise, the completeness results will be too optimistic.

     Please check the parameters used to run Aegean inside the script. They should be exactly the same as the parameters used to generate the final source catalogue. 
     In this example, I set the source detection limit to 4 sigma, but you should adjust that as required.

4) Calculate the overall completeness as a function of flux and generate completeness maps (one per flux level)

     Command line example:
     sbatch --time=05:00:00 make_cmp_map.sh injected_sources=*/source_pos/source_pos.txt detected_sources=*/inject flux=-2,0,0.1 template_map=*/input_images/JDGP_wideband_projpsf_psf.fits
     region=175,290,-70,10 rad=6 output_dir=*/results
   
     The input parameters are explained at the top of the script.
     I set rad=6, but you can increase this if you want to apply smoother to the completeness maps.
     Creating the completeness maps may take several hours as it involves calculating many distances on a sphere.
