.. :changelog:

Changes
*******

.. **DEFINE LATEST CHANGES UNDER BELOW 'Unreleased' SECTION - THEY WILL BE INTEGRATED IN NEXT RELEASE**
`Unreleased <https://github.com/bird-house/birdhouse-deploy/tree/master>`_ (latest)
========================================================================================================================

- <changes here> ...

`1.12.1 <https://github.com/bird-house/birdhouse-deploy/tree/1.12.1>`_ (2021-04-28)
========================================================================================================================

- proxy: allow homepage (location /) to be configurable

`1.12.0 <https://github.com/bird-house/birdhouse-deploy/tree/1.12.0>`_ (2021-04-19)
========================================================================================================================

- Magpie upgrade strike II
  
  Strike II of this original PR https://github.com/bird-house/birdhouse-deploy/pull/107.
  
  Matching notebook fix https://github.com/Ouranosinc/pavics-sdi/pull/218
  
  Performed test upgrade on staging (Medus) using prod (Boreas) Magpie DB, everything went well and Jenkins passed (http://jenkins.ouranos.ca/job/ouranos-staging/job/medus.ouranos.ca/80/parameters/).  This Jenkins build uses the corresponding branch in https://github.com/Ouranosinc/pavics-sdi/pull/218 and with `TEST_MAGPIE_AUTH` enabled.
  
  Manual upgrade migration procedure:
  1. Save `/data/magpie_persist` folder from prod `pavics.ouranos.ca`: `cd /data; tar czf magpie_persist.prod.tgz magpie_persist`
  2. scp `magpie_persist.prod.tgz` to `medus`
  3. login to `medus`
  4. `cd /path/to/birdhouse-deploy/birdhouse`
  3. `./pavics-compose.sh down`
  4. `git checkout master`
  5. `cd /data`
  2. `rm -rf magpie_persist`
  3. `tar xzf magpie_persist.prod.tgz`  # restore Magpie DB with prod version
  4. `cd /path/to/birdhouse-deploy/birdhouse`
  5. `./pavics-compose.sh up -d`
  3. Update `env.local` `MAGPIE_ADMIN_PASSWORD` with prod passwd for Twitcher to be able to access Magpie since we juste restore the Magpie DB from prod
  4. `./pavics-compose.sh restart twitcher`  # for Twitcher to get new Magpie admin passwd 
  4. Baseline working state: trigger Jenkins test suite, ensure all pass except `pavics_thredds.ipynb` that requires new Magpie
  5. Baseline working state: view existing services permissions on group Anonymous (https://medus.ouranos.ca/magpie/ui/groups/anonymous/default)
  6. `git checkout restore-previous-broken-magpie-upgrade-so-we-can-work-on-a-fix`  # This current branch
  7. `./pavics-compose.sh up -d`  # upgrade to new Magpie
  8. `docker logs magpie`: check no DB migration error
  9. Trigger Jenkins test suite again

`1.11.29 <https://github.com/bird-house/birdhouse-deploy/tree/1.11.29>`_ (2021-04-16)
========================================================================================================================

- Update Raven and Jupyter env for Raven demo
  
  Raven release notes PR https://github.com/Ouranosinc/raven/pull/374 + https://github.com/Ouranosinc/raven/pull/382
  
  Jupyter env update PR https://github.com/Ouranosinc/PAVICS-e2e-workflow-tests/pull/71
  
  Other fixes:
  * Fix intermittent Jupyter spawning error by doubling various timeouts config (it's intermittent so hard to test so we are not sure which ones of timeout fixed it)
  * Fix Finch and Raven "Broken pipe" error when the request size is larger than default 3mb (bumped to 100mb) (fixes https://github.com/Ouranosinc/raven/issues/361 and Finch related comment https://github.com/bird-house/finch/issues/98#issuecomment-811230388)
  * Lower chance to have "Max connection" error for Finch and Raven (bump parallelprocesses from 2 to 10). In prod, the server has the CPU needed to run 10 concurrent requests if needed so this prevent users having to "wait" after each other.

`1.11.28 <https://github.com/bird-house/birdhouse-deploy/tree/1.11.28>`_ (2021-04-09)
========================================================================================================================

- jupyter: update for new clisops, xclim, ravenpy
  
  Matching PR to deploy the new Jupyter env to PAVICS.
  
  See PR https://github.com/Ouranosinc/PAVICS-e2e-workflow-tests/pull/68
  for more info.
  
  Relevant changes:
  ```diff
  <   - clisops=0.5.1=pyhd3deb0d_0
  >   - clisops=0.6.3=pyh44b312d_0
  
  <   - xclim=0.23.0=pyhd8ed1ab_0
  >   - xclim=0.25.0=pyhd8ed1ab_0
  
  >   - ostrich=0.1.2=h2bc3f7f_0
  >   - raven=0.1.1=h2bc3f7f_0
  
  <     - ravenpy==0.2.3  # from pip
  >   - ravenpy=0.3.1=py37_0  # from conda
  
  >   - aiohttp=3.7.4=py37h5e8e339_0
  
  <   - roocs-utils=0.1.5=pyhd3deb0d_1
  >   - roocs-utils=0.3.0=pyh6c4a22f_0
  
  <   - cf_xarray=0.4.0=pyh44b312d_0
  >   - cf_xarray=0.5.1=pyh44b312d_0
  
  <   - rioxarray=0.2.0=pyhd8ed1ab_0
  >   - rioxarray=0.3.1=pyhd8ed1ab_0
  
  <   - xarray=0.16.2=pyhd8ed1ab_0
  >   - xarray=0.17.0=pyhd8ed1ab_0
  
  <   - geopandas=0.8.2=pyhd8ed1ab_0
  >   - geopandas=0.9.0=pyhd8ed1ab_0
  
  <   - gdal=3.1.4=py37h2ec2946_5
  >   - gdal=3.2.1=py37hc5bc4e4_7
  
  <   - jupyter_conda=4.1.0=hd8ed1ab_1
  >   - jupyter_conda=5.0.0=hd8ed1ab_0
  
  <   - python=3.7.9=hffdb5ce_100_cpython
  >   - python=3.7.10=hffdb5ce_100_cpython
  ```

`1.11.27 <https://github.com/bird-house/birdhouse-deploy/tree/1.11.27>`_ (2021-04-01)
========================================================================================================================

- reverted name of monitoring routes to original
  
  The Canarie API complains that `stats` are up but don't return the correct response. I assume it's because I changed the monitoring key to reflect the actual content. 
  
  Validation service: https://science.canarie.ca/researchsoftware/services/validator/service.html
  Use Managed
  Enter URL: http://pavics.ouranos.ca/canarie/renderer
  Submit
  
  Deployed on my dev VM, fix worked, thanks !
  
  ![Screenshot_2021-04-01 CANARIE Research Software Logiciels de recherche de CANARIE](https://user-images.githubusercontent.com/11966697/113295664-7c359b80-92c6-11eb-8c50-5e77f498d84f.png)

`1.11.26 <https://github.com/bird-house/birdhouse-deploy/tree/1.11.26>`_ (2021-03-31)
========================================================================================================================

- Update canarieAPI doc links
  
  - Updated components' version number. 
  - Replaced links to githubio docs to readthedocs.
  - renderer is provided by THREDDS-WMS. 
  - slicer is provided by finch.

`1.11.25 <https://github.com/bird-house/birdhouse-deploy/tree/1.11.25>`_ (2021-03-26)
========================================================================================================================

- finch: update to version 0.7.1
  
  See Finch release PR https://github.com/bird-house/finch/pull/164 for more release info.
  
  This update will fix the following Jenkins error introduced by https://github.com/bird-house/finch/pull/161#discussion_r601975311:
  
  ```
  12:37:00  _________ finch-master/docs/source/notebooks/finch-usage.ipynb::Cell 1 _________
  12:37:00  Notebook cell execution failed
  12:37:00  Cell 1: Cell outputs differ
  12:37:00
  12:37:00  Input:
  12:37:00  help(wps.frost_days)
  12:37:00
  12:37:00  Traceback:
  12:37:00   mismatch 'stdout'
  12:37:00
  12:37:00   assert reference_output == test_output failed:
  12:37:00
  12:37:00    'Help on meth...ut files.\n\n' == 'Help on meth...ut files.\n\n'
  12:37:00    Skipping 70 identical leading characters in diff, use -v to show
  12:37:00    - min=None, missing_options=None, check_missing='any', thresh='0 degC', freq='YS', variable=None, output_formats=None) method of birdy.client.base.WPSClient instance
  12:37:00    + min=None, check_missing='any', cf_compliance='warn', data_validation='raise', thresh='0 degC', freq='YS', missing_options=None, variable=None, output_formats=None) method of birdy.client.base.WPSClient instance
  12:37:00          Number of days where daily minimum temperatures are below 0.
  12:37:00
  12:37:00          Parameters
  12:37:00          ----------
  12:37:00          tasmin : ComplexData:mimetype:`application/x-netcdf`, :mimetype:`application/x-ogc-dods`
  12:37:00              NetCDF Files or archive (tar/zip) containing netCDF files.
  12:37:00          thresh : string
  12:37:00              Freezing temperature.
  12:37:00          freq : {'YS', 'MS', 'QS-DEC', 'AS-JUL'}string
  12:37:00              Resampling frequency.
  12:37:00          check_missing : {'any', 'wmo', 'pct', 'at_least_n', 'skip', 'from_context'}string
  12:37:00              Method used to determine which aggregations should be considered missing.
  12:37:00          missing_options : ComplexData:mimetype:`application/json`
  12:37:00              JSON representation of dictionary of missing method parameters.
  12:37:00    +     cf_compliance : {'log', 'warn', 'raise'}string
  12:37:00    +         Whether to log, warn or raise when inputs have non-CF-compliant attributes.
  12:37:00    +     data_validation : {'log', 'warn', 'raise'}string
  12:37:00    +         Whether to log, warn or raise when inputs fail data validation checks.
  12:37:00          variable : string
  12:37:00              Name of the variable in the NetCDF file.
  12:37:00
  12:37:00          Returns
  12:37:00          -------
  12:37:00          output_netcdf : ComplexData:mimetype:`application/x-netcdf`
  12:37:00              The indicator values computed on the original input grid.
  12:37:00          output_log : ComplexData:mimetype:`text/plain`
  12:37:00              Collected logs during process run.
  12:37:00          ref : ComplexData:mimetype:`application/metalink+xml; version=4.0`
  12:37:00              Metalink file storing all references to output files.
  ```
  
  Jenkins build with Finch notebooks passing against newer Finch: http://jenkins.ouranos.ca/job/ouranos-staging/job/lvupavics.ouranos.ca/45/console

`1.11.24 <https://github.com/bird-house/birdhouse-deploy/tree/1.11.24>`_ (2021-03-19)
========================================================================================================================

- Avoid docker pull since pull rate limit on dockerhub
  
  Pin bash tag so it is reproducible (previously it was more or less reproducible since we always ensure "latest" tag).
  
  Avoid the following error:
  
  ```
  + docker pull bash
  Using default tag: latest
  Error response from daemon: toomanyrequests: You have reached your pull rate limit. You may increase the limit by authenticating and upgrading: https://www.docker.com/increase-rate-limit
  ```

`1.11.23 <https://github.com/bird-house/birdhouse-deploy/tree/1.11.23>`_ (2021-03-17)
========================================================================================================================

- Custom Jupyter user images
  
  Adds CRIM's nlp and eo images to the available list of images in JupyterHub
  
  The base image (pavics-jupyter-base) wasn't added to the list, because it is assumed the users will always be using the other more specialized images.
  
  We were already able to add/override Jupyter images but this PR makes it more integrated: those image will also be pulled in advanced so startup is much faster for big images since these images will already be cached.
  
  Backward incompatible changes:
  DOCKER_NOTEBOOK_IMAGE renamed to DOCKER_NOTEBOOK_IMAGES and is now a space separated list of images. Any existing override in env.local using the old name will have to switch to the new name.

`1.11.22 <https://github.com/bird-house/birdhouse-deploy/tree/1.11.22>`_ (2021-03-16)
========================================================================================================================

- finch: update to 0.7.0
  
  Require PR https://github.com/bird-house/birdhouse-deploy/pull/131 for extra testdata for the new regridding notebook.
  
  Regridding notebook will also need to be adjusted for some output to pass Jenkins test suite, PR https://github.com/Ouranosinc/pavics-sdi/pull/206.
  
  Nbval escape regex also needed for the regridding notebook, PR https://github.com/Ouranosinc/PAVICS-e2e-workflow-tests/pull/63
  
  See Finch changelog in PR https://github.com/bird-house/finch/pull/158
  
  Passing Jenkins build http://jenkins.ouranos.ca/job/PAVICS-e2e-workflow-tests/job/update-nbval-sanitize-config-for-pavics-sdi-regridding-notebook/10/console

`1.11.21 <https://github.com/bird-house/birdhouse-deploy/tree/1.11.21>`_ (2021-02-19)
========================================================================================================================

- Configurable Jupyterhub README
  
  While the [`README.ipynb`](https://github.com/bird-house/birdhouse-deploy/blob/master/docs/source/notebooks/README.ipynb) provided by `birdhouse-deploy` is good, it does not quite fit our needs at PCIC. This PR allows users to configure their own `README` for Jupyterhub.
  
  ## Changes
  - Adds `JUPYERHUB_README` as configuration option in the appropriate spots

`1.11.20 <https://github.com/bird-house/birdhouse-deploy/tree/1.11.20>`_ (2021-02-19)
========================================================================================================================

- jupyter: update to version 210216 for xESMF
  
  Matching PR to deploy https://github.com/Ouranosinc/PAVICS-e2e-workflow-tests/pull/61 to PAVICS.
  
  For regridding notebook, see https://github.com/Ouranosinc/pavics-sdi/pull/201#issuecomment-778329309.
  
  Noticeable changes:
  
  ```diff
  >   - xesmf=0.5.2=pyhd8ed1ab_0
  
  <   - owslib=0.21.0=pyhd8ed1ab_0
  >   - owslib=0.23.0=pyhd8ed1ab_0
  
  <   - cftime=1.3.1=py37h6323ea4_0
  >   - cftime=1.4.1=py37h902c9e0_0
  
  <   - dask=2021.1.1=pyhd8ed1ab_0
  >   - dask=2021.2.0=pyhd8ed1ab_0
  
  <   - rioxarray=0.1.1=pyhd8ed1ab_0
  >   - rioxarray=0.2.0=pyhd8ed1ab_0
  ```

`1.11.19 <https://github.com/bird-house/birdhouse-deploy/tree/1.11.19>`_ (2021-02-10)
========================================================================================================================

- proxy: proxy_read_timeout config should be configurable
  
  We have a performance problem with the production deployment at Ouranos so we need a longer timeout.  Being a Ouranos specific need, it should not be hardcoded as in previous PR https://github.com/bird-house/birdhouse-deploy/pull/122.
  
  The previous increase was sometime not enough !
  
  The value is now configurable via `env.local` as most other customizations.  Documentation updated.
  
  Timeout in Prod:
  ```
  WPS_URL=https://pavics.ouranos.ca/twitcher/ows/proxy/raven/wps FINCH_WPS_URL=https://pavics.ouranos.ca/twitcher/ows/proxy/finch/wps FLYINGPIGEON_WPS
  _URL=https://pavics.ouranos.ca/twitcher/ows/proxy/flyingpigeon/wps pytest --nbval-lax --verbose docs/source/notebooks/Running_HMETS_with_CANOPEX_datas
  et.ipynb --sanitize-with docs/source/output-sanitize.cfg --ignore docs/source/notebooks/.ipynb_checkpoints
  
  HTTPError: 504 Server Error: Gateway Time-out for url: https://pavics.ouranos.ca/twitcher/ows/proxy/raven/wps
  
  ===================================================== 11 failed, 4 passed, 1 warning in 249.80s (0:04:09) ===========================================
  ```
  
  Pass easily on my test VM with very modest hardware (10G ram, 2 cpu):
  ```
  WPS_URL=https://lvupavicsmaster.ouranos.ca/twitcher/ows/proxy/raven/wps FINCH_WPS_URL=https://lvupavicsmaster.ouranos.ca/twitcher/ows/proxy/finch/wp
  s FLYINGPIGEON_WPS_URL=https://lvupavicsmaster.ouranos.ca/twitcher/ows/proxy/flyingpigeon/wps pytest --nbval-lax --verbose docs/source/notebooks/Runni
  ng_HMETS_with_CANOPEX_dataset.ipynb --sanitize-with docs/source/output-sanitize.cfg --ignore docs/source/notebooks/.ipynb_checkpoints
  
  =========================================================== 15 passed, 1 warning in 33.84s ===========================================================
  ```
  
  Pass against Medus:
  ```
  WPS_URL=https://medus.ouranos.ca/twitcher/ows/proxy/raven/wps FINCH_WPS_URL=https://medus.ouranos.ca/twitcher/ows/proxy/finch/wps FLYINGPIGEON_WPS_URL=https://medus.ouranos.ca/twitcher/ows/proxy/flyingpigeon/wps pytest --nbval-lax --verbose docs/source/notebooks/Running_HMETS_with_CANOPEX_dataset.ipynb --sanitize-with docs/source/output-sanitize.cfg --ignore docs/source/notebooks/.ipynb_checkpoints
  
  ============================================== 15 passed, 1 warning in 42.44s =======================================================
  ```
  
  Pass against `hirondelle.crim.ca`:
  ```
  WPS_URL=https://hirondelle.crim.ca/twitcher/ows/proxy/raven/wps FINCH_WPS_URL=https://hirondelle.crim.ca/twitcher/ows/proxy/finch/wps FLYINGPIGEON_WPS_URL=https://hirondelle.crim.ca/twitcher/ows/proxy/flyingpigeon/wps pytest --nbval-lax --verbose docs/source/notebooks/Running_HMETS_with_CANOPEX_dataset.ipynb --sanitize-with docs/source/output-sanitize.cfg --ignore docs/source/notebooks/.ipynb_checkpoints
  
  =============================================== 15 passed, 1 warning in 35.61s ===============================================
  ```
  
  For comparison, a run on Prod without Twitcher (PR https://github.com/bird-house/birdhouse-deploy-ouranos/pull/5):
  ```
  WPS_URL=https://pavics.ouranos.ca/raven/wps FINCH_WPS_URL=https://pavics.ouranos.ca/twitcher/ows/proxy/finch/wps FLYINGPIGEON_WPS_URL=https://pavics
  .ouranos.ca/twitcher/ows/proxy/flyingpigeon/wps pytest --nbval-lax --verbose docs/source/notebooks/Running_HMETS_with_CANOPEX_dataset.ipynb --sanitize
  -with docs/source/output-sanitize.cfg --ignore docs/source/notebooks/.ipynb_checkpoints
  
  HTTPError: 504 Server Error: Gateway Time-out for url: https://pavics.ouranos.ca/raven/wps
  
  ================================================ 11 failed, 4 passed, 1 warning in 248.99s (0:04:08) =================================================
  ```
  
  A run on Prod without Twitcher and Nginx (direct hit Raven):
  ```
  WPS_URL=http://pavics.ouranos.ca:8096/ FINCH_WPS_URL=https://pavics.ouranos.ca/twitcher/ows/proxy/finch/wps FLYINGPIGEON_WPS_URL=https://pavics.oura
  nos.ca/twitcher/ows/proxy/flyingpigeon/wps pytest --nbval-lax --verbose docs/source/notebooks/Running_HMETS_with_CANOPEX_dataset.ipynb --sanitize-with
   docs/source/output-sanitize.cfg --ignore docs/source/notebooks/.ipynb_checkpoints
  
  ===================================================== 15 passed, 1 warning in 218.46s (0:03:38) ======================================================

`1.11.18 <https://github.com/bird-house/birdhouse-deploy/tree/1.11.18>`_ (2021-02-02)
========================================================================================================================

- update Raven and Jupyter env
  
  See https://github.com/Ouranosinc/raven/compare/v0.10.0...v0.11.1 for change details.
  
  Jupyter env change details: https://github.com/Ouranosinc/PAVICS-e2e-workflow-tests/pull/60
  
  Jenkins run (this Jupyter env `pavics/workflow-tests:210201.2` against a devel version of Raven `0.11.1` + `--nbval-lax`) http://jenkins.ouranos.ca/job/PAVICS-e2e-workflow-tests/job/test-nbval-lax-DO_NOT_MERGE/4/console
  
  Only known error:
  ```
  20:25:45  =========================== short test summary info ============================
  20:25:45  FAILED pavics-sdi-master/docs/source/notebooks/WMS_example.ipynb::Cell 1
  20:25:45  FAILED pavics-sdi-master/docs/source/notebooks/WMS_example.ipynb::Cell 2
  20:25:45  FAILED pavics-sdi-master/docs/source/notebooks/WMS_example.ipynb::Cell 3
  20:25:45  FAILED pavics-sdi-master/docs/source/notebooks/WMS_example.ipynb::Cell 4
  20:25:45  FAILED pavics-sdi-master/docs/source/notebooks/WMS_example.ipynb::Cell 5
  20:25:45  FAILED pavics-sdi-master/docs/source/notebooks/WMS_example.ipynb::Cell 6
  20:25:45  FAILED pavics-sdi-master/docs/source/notebooks/WMS_example.ipynb::Cell 7
  20:25:45  FAILED raven-master/docs/source/notebooks/Bias_correcting_climate_data.ipynb::Cell 8
  20:25:45  FAILED raven-master/docs/source/notebooks/Bias_correcting_climate_data.ipynb::Cell 9
  20:25:45  FAILED raven-master/docs/source/notebooks/Bias_correcting_climate_data.ipynb::Cell 10
  20:25:45  FAILED raven-master/docs/source/notebooks/Bias_correcting_climate_data.ipynb::Cell 11
  20:25:45  FAILED raven-master/docs/source/notebooks/Full_process_example_1.ipynb::Cell 13
  20:25:45  FAILED raven-master/docs/source/notebooks/Full_process_example_1.ipynb::Cell 17
  20:25:45  FAILED raven-master/docs/source/notebooks/Full_process_example_1.ipynb::Cell 18
  20:25:45  FAILED raven-master/docs/source/notebooks/Full_process_example_1.ipynb::Cell 19
  20:25:45  FAILED raven-master/docs/source/notebooks/Full_process_example_1.ipynb::Cell 20
  20:25:45  FAILED raven-master/docs/source/notebooks/Full_process_example_1.ipynb::Cell 21
  20:25:45  FAILED raven-master/docs/source/notebooks/Multiple_watersheds_simulation.ipynb::Cell 1
  20:25:45  FAILED raven-master/docs/source/notebooks/Multiple_watersheds_simulation.ipynb::Cell 3
  20:25:45  FAILED raven-master/docs/source/notebooks/Multiple_watersheds_simulation.ipynb::Cell 4
  20:25:45  FAILED raven-master/docs/source/notebooks/Multiple_watersheds_simulation.ipynb::Cell 5
  20:25:45  FAILED raven-master/docs/source/notebooks/Region_selection.ipynb::Cell 7
  20:25:45  FAILED raven-master/docs/source/notebooks/Region_selection.ipynb::Cell 8
  20:25:45  FAILED raven-master/docs/source/notebooks/Subset_climate_data_over_watershed.ipynb::Cell 5
  20:25:45  ============ 24 failed, 226 passed, 2 skipped in 2528.69s (0:42:08) ============
  ```

`1.11.17 <https://github.com/bird-house/birdhouse-deploy/tree/1.11.17>`_ (2021-01-28)
========================================================================================================================

- finch: update to version 0.6.1
  
  See Finch PR https://github.com/bird-house/finch/pull/147 for release notes.
  
  Deployed on my dev server, Jenkins run no new errors: http://jenkins.ouranos.ca/job/PAVICS-e2e-workflow-tests/job/master/900/console

`1.11.16 <https://github.com/bird-house/birdhouse-deploy/tree/1.11.16>`_ (2021-01-14)
========================================================================================================================

- finch: upgrade to version 0.6.0
  
  See Finch PR for release notes https://github.com/bird-house/finch/pull/138.
  
  Should fix https://github.com/Ouranosinc/PAVICS-e2e-workflow-tests/issues/58.

`1.11.15 <https://github.com/bird-house/birdhouse-deploy/tree/1.11.15>`_ (2021-01-14)
========================================================================================================================

- jupyter: update to version 201214
  
  Matching PR to deploy the new Jupyter env in PR https://github.com/Ouranosinc/PAVICS-e2e-workflow-tests/pull/56 to PAVICS.
  
  Relevant changes:
  
  ```diff
  >   - cfgrib=0.9.8.5=pyhd8ed1ab_0
  
  <   - clisops=0.3.1=pyh32f6830_1
  >   - clisops=0.4.0=pyhd3deb0d_0
  
  <   - dask=2.30.0=py_0
  >   - dask=2020.12.0=pyhd8ed1ab_0
  
  <   - owslib=0.20.0=py_0
  >   - owslib=0.21.0=pyhd8ed1ab_0
  
  <   - xarray=0.16.1=py_0
  >   - xarray=0.16.2=pyhd8ed1ab_0
  
  <   - xclim=0.21.0=py_0
  >   - xclim=0.22.0=pyhd8ed1ab_0
  
  <   - jupyter_conda=3.4.1=pyh9f0ad1d_0
  >   - jupyter_conda=4.1.0=hd8ed1ab_1
  ```

`1.11.14 <https://github.com/bird-house/birdhouse-deploy/tree/1.11.14>`_ (2020-12-17)
========================================================================================================================

- Add ability to execute post actions for deploy-data script.
  
  Script `deploy-data` was previously introduced in PR https://github.com/bird-house/birdhouse-deploy/pull/72 to deploy any files from any git repos to the local host it runs.
  
  Now it grows the ability to run commands from the git repo it just pulls.
  
  Being able to run commands open new possibilities:
  * post-processing after files from git repo are deployed (ex: advanced file re-mapping)
  * execute up-to-date scripts from git repos (PR https://github.com/bird-house/birdhouse-deploy-ouranos/pull/2)
  
  Combining this `deploy-data` with the `scheduler` component means we have a way for cronjobs to automatically always execute the most up-to-date version of any scripts from any git repos.

`1.11.13 <https://github.com/bird-house/birdhouse-deploy/tree/1.11.13>`_ (2020-12-14)
========================================================================================================================

- jupyterhub: update to version 1.3.0 to include login terms patch
  
  This version of jupyterhub includes the login terms patch originally
  introduced in commit 8be8eeac211d3f5c2de620781db8832fdb8f9093 (PR https://github.com/bird-house/birdhouse-deploy/pull/104).
  
  This official login terms feature has a few enhancements (see https://github.com/jupyterhub/jupyterhub/pull/3264#discussion_r530466178):
  
  * no javascript dependency
  * pop-up reminder for user to check the checkbox
  
  Behavior change is the "Sign in" button is not longer disabled if
  unchecked.  It simply does not work and reminds the user to check the
  checkbox if unchecked.
  
  Before:
  
  ![recorded](https://user-images.githubusercontent.com/11966697/99327962-1aa9ee80-2849-11eb-9ce8-3be6a1484e94.gif)
  
  After:
  ![recorded](https://user-images.githubusercontent.com/11966697/100246404-18115e00-2f07-11eb-9061-d35434ace3aa.gif)

`1.11.12 <https://github.com/bird-house/birdhouse-deploy/tree/1.11.12>`_ (2020-11-25)
========================================================================================================================

- Fix geoserver not configured properly behind proxy.
  
  Hitting https://pavics.ouranos.ca/geoserver/wfs?request=GetCapabilities&version=1.1.0
  
  Before fix (wrong scheme and wrong port):
  ```
  <ows:Operation name="GetCapabilities">
  <ows:DCP>
  <ows:HTTP>
  <ows:Get xlink:href="http://pavics.ouranos.ca:80/geoserver/wfs"/>
  <ows:Post xlink:href="http://pavics.ouranos.ca:80/geoserver/wfs"/>
  </ows:HTTP>
  </ows:DCP>
  ```
  
  After fix:
  ```
  <ows:Operation name="GetCapabilities">
  <ows:DCP>
  <ows:HTTP>
  <ows:Get xlink:href="https://pavics.ouranos.ca:443/geoserver/wfs"/>
  <ows:Post xlink:href="https://pavics.ouranos.ca:443/geoserver/wfs"/>
  </ows:HTTP>
  </ows:DCP>
  ```
  
  This config automate manual step to set proxy base url in Geoserver UI https://docs.geoserver.org/2.9.3/user/configuration/globalsettings.html#proxy-base-url
  
  I had to override the docker image entrypoint to edit the `server.xml` on the fly before starting Geoserver (Tomcat) since setting Java proxy config did not seem to work (see first commit).
  
  Related to https://github.com/Ouranosinc/raven/issues/297.

`1.11.11 <https://github.com/bird-house/birdhouse-deploy/tree/1.11.11>`_ (2020-11-20)
========================================================================================================================

- Various small fixes.
  
  monitoring: prevent losing stats when VM auto start from a power failure
  
  check-instance-ready: new script to smoke test instance (use in `bootstrap-instance-for-testsuite` for our automation pipeline).
  
  jupyter: add CATALOG_USERNAME and anonymous to blocked_users list for security
  See comment https://github.com/bird-house/birdhouse-deploy/pull/102#issuecomment-730109547
  and comment https://github.com/bird-house/birdhouse-deploy/pull/102#issuecomment-730407914
      
      They are not real Jupyter users and their password is known.
      
      See config/magpie/permissions.cfg.template that created those users.
      
      Tested:
      ```
      [W 2020-11-20 13:25:18.924 JupyterHub auth:487] User 'admin-catalog' blocked. Stop authentication
      [W 2020-11-20 13:25:18.924 JupyterHub base:752] Failed login for admin-catalog
      
      [W 2020-11-20 13:49:18.069 JupyterHub auth:487] User 'anonymous' blocked. Stop authentication
      [W 2020-11-20 13:49:18.070 JupyterHub base:752] Failed login for anonymous
      ```

`1.11.10 <https://github.com/bird-house/birdhouse-deploy/tree/1.11.10>`_ (2020-11-18)
========================================================================================================================

- Add terms conditions to JupyterHub login page and update to latest JupyterHub version.
  
  User have to check the checkbox agreeing to the terms and conditions in order to login (fixes https://github.com/Ouranosinc/pavics-sdi/issues/188).
  
  User will have to accept the terms and conditions (the checkbox) each time he needs to login. However, if user do not logout or wipe his browser cookies, the next time he navigate to the login page, he'll just log right in, no password is asked so no terms and conditions to accept either.
  
  This behavior is optional and only enabled if `JUPYTER_LOGIN_TERMS_URL` in `env.local` is set.
  
  Had to patch the `login.html` template from jupyterhub docker image for this feature (PR https://github.com/jupyterhub/jupyterhub/pull/3264).
  
  Also update jupyterhub docker image to latest version.
  
  Deployed to my test server https://lvupavics.ouranos.ca/jupyter/hub/login (pointing to a bogus terms and conditions link for now).
  
  Tested on Firefox and Google Chrome.
  
  Tested that upgrade from jupyterhub `1.0.0` to `1.2.1` is completely transparent to already logged in jupyter users.
  ```
  [D 2020-11-18 19:53:52.517 JupyterHub app:2055] Verifying that lvu is running at http://172.18.0.3:8888/jupyter/user/lvu/                             
  [D 2020-11-18 19:53:52.523 JupyterHub utils:220] Server at http://172.18.0.3:8888/jupyter/user/lvu/ responded with 302                                
  [D 2020-11-18 19:53:52.523 JupyterHub _version:76] jupyterhub and jupyterhub-singleuser both on version 1.2.1                                         
  [I 2020-11-18 19:53:52.524 JupyterHub app:2069] lvu still running    
  ```
  
  ![recorded](https://user-images.githubusercontent.com/11966697/99327962-1aa9ee80-2849-11eb-9ce8-3be6a1484e94.gif)

`1.11.9 <https://github.com/bird-house/birdhouse-deploy/tree/1.11.9>`_ (2020-11-13)
========================================================================================================================

- jupyter: new image with 4 new extensions
  
  The google drive extension for JupyterLab requires a settings file containing the clientid of the project created in developers.google.com, which give authorization to use google drive.
  
  This PR's role is to include this file in the birdhouse configs.
  
  Matching PR https://github.com/Ouranosinc/PAVICS-e2e-workflow-tests/pull/54 (commit https://github.com/Ouranosinc/PAVICS-e2e-workflow-tests/commit/5d5a9aa2251386378406efb5b414b3aa6db0b37e) for the new image with 4 new extensions: jupytext, jupyterlab-google-drive, jupyter_conda and jupyterlab-git
  
  Matching PR https://github.com/Ouranosinc/pavics-sdi/pull/185 for documention about the new extensions.

`1.11.8 <https://github.com/bird-house/birdhouse-deploy/tree/1.11.8>`_ (2020-11-06)
========================================================================================================================

- bump `finch` to version-0.5.3

`1.11.7 <https://github.com/bird-house/birdhouse-deploy/tree/1.11.7>`_ (2020-11-06)
========================================================================================================================

- bump `thredds-docker` to 4.6.15

`1.11.6 <https://github.com/bird-house/birdhouse-deploy/tree/1.11.6>`_ (2020-11-06)
========================================================================================================================

- Prepare fresh deployment for automated tests.
  
  @MatProv is building an automated pipeline that will provision and deploy a full PAVICS stack and run our Jenkins test suite for each PR here.
  
  So each time his new fresh instance comes up, there are a few steps to perform for the Jenkins test suite to pass.  Those steps are captured in `scripts/bootstrap-instance-for-testsuite`.  @MatProv please call this script, do not perform each steps yourself so any future changes to those steps will be transparent to your pipeline.  A new optional components was also required, done in PR https://github.com/bird-house/birdhouse-deploy/pull/92.
  
  For security reasons, Jupyterhub will block the test user to login since its password is known publicly.
  
  Each step are also in their own script so can be assembled differently to prepare the fresh instance if desired.
  
  Solr query in the canarie monitoring also updated to target the minimal dataset from `bootstrap-testdata` so the canarie monitoring page works on all PAVICS deployment (fixes https://github.com/bird-house/birdhouse-deploy/issues/6).  @MatProv you can use this canarie monitoring page (ex: https://pavics.ouranos.ca/canarie/node/service/status) to confirm the fresh instance is ready to run the Jenkins test suite.

`1.11.5 <https://github.com/bird-house/birdhouse-deploy/tree/1.11.5>`_ (2020-10-27)
========================================================================================================================

- jupyter: new image for python 3.8, new xclim and memory_profiler
  
  Matching PR to deploy the new Jupyter image to PAVICS.
  
  Deployed to https://medus.ouranos.ca/jupyter/ for testing.  This one has python 3.8, might worth some manual testing.
  
  Relevant changes:
  ```diff
  <   - python=3.7.8=h6f2ec95_1_cpython
  >   - python=3.8.6=h852b56e_0_cpython
  
  <   - xclim=0.20.0=py_0
  >   - xclim=0.21.0=py_0
  
  <   - dask=2.27.0=py_0
  >   - dask=2.30.0=py_0
  
  <   - rioxarray=0.0.31=py_0
  >   - rioxarray=0.1.0=py_0
  
  >   - memory_profiler=0.58.0=py_0
  ```
  
  More info, see PR https://github.com/Ouranosinc/PAVICS-e2e-workflow-tests/pull/53 (commit https://github.com/Ouranosinc/PAVICS-e2e-workflow-tests/commit/f07f1657ed13a0ed92854c5d01f9d3ed785e870d)

`1.11.4 <https://github.com/bird-house/birdhouse-deploy/tree/1.11.4>`_ (2020-10-15)
========================================================================================================================

- Sync Raven testdata to Thredds for Raven tutorial notebooks.
  
  Leveraging the cron daemon of the scheduler component, sync Raven testdata to Thredds for Raven tutorial notebooks.
  
  Activation of the pre-configured cronjob is via `env.local` as usual for infra-as-code.
  
  New generic `deploy-data` script can clone any number of git repos, sync any number of folders in the git repo to any number of local folders, with ability to cherry-pick just the few files needed (Raven testdata has many types of files, we only need to sync `.nc` files to Thredds, to avoid polluting Thredds storage `/data/datasets/testdata/raven`).
  
  Limitation of the first version of this `deploy-data` script:
  * Do not handle re-organizing file layout, this is a pure sync only with very limited rsync filtering for now (tutorial notebooks deploy from multiple repos, need re-organizing the file layout)
  
  So the script has room to grow.  I see it as a generic solution to the repeated problem "take files from various git repos and deploy them somewhere automatically".  If we need to deploy another repo, juste write a new config file, stop writing boilerplate code again.
  
  Minor unrelated change in this PR:
  * README update to reference the new birdhouse-deploy-ouranos.
  * Make sourcing the various pre-configured cronjob backward-compat with older version of the repo where those cronjob did not exist yet.

`1.11.3 <https://github.com/bird-house/birdhouse-deploy/tree/1.11.3>`_ (2020-09-28)
========================================================================================================================

- jupyter: new build for new xclim with fix for missing clisops dependency
  
  Matching PR to deploy new Jupyter env from PR https://github.com/Ouranosinc/PAVICS-e2e-workflow-tests/pull/52 (commit https://github.com/Ouranosinc/PAVICS-e2e-workflow-tests/commit/18c8397ff30c9ba4b9f56896df4c898c7e9a356e).
  
  Deployed to https://medus.ouranos.ca/jupyter/ for testing.
  
  Relevant changes:
  ```diff
  >   - clisops=0.3.1=pyh32f6830_1
  
  <     - xclim==0.19.0
  >   - xclim=0.20.0=py_0
  
  <   - xarray=0.16.0py_0
  >   - xarray=0.16.1=py_0
  
  <   - dask=2.26.0=py_0
  >   - dask=2.27.0=py_0
  
  <   - fiona=1.8.13=py37h0492a4a_1
  >   - fiona=1.8.17=py37ha3d844c_0
  
  <   - gdal=3.0.4=py37h4b180d9_10
  >   - gdal=3.1.2=py37h518339e_2
  
  <   - jupyter_server=0.1.1=py37_0
  >   - jupyter_server=1.0.1=py37hc8dfbb8_0
  
  >     - jupyternotify==0.1.15
  
  >     - pytest-tornasync==0.6.0.post2
  ```
  
  See PR above for full changes.

`1.11.2 <https://github.com/bird-house/birdhouse-deploy/tree/1.11.2>`_ (2020-09-15)
========================================================================================================================

- Auto-renew LetsEncrypt SSL certificate.
  
  Auto-renew LetsEncrypt SSL certificate leveraging the cron jobs of the "scheduler" component.  Meaning this feature is self-contained in the PAVICS stack, no dependency on the host's cron jobs.
  
  Default behavior is to attempt renewal everyday.  `certbot` client in `renew` mode will not hit LetsEncrypt server if renewal is not allowed (not within 1 month of expiry) so this should not put too much stress on LetsEncrypt server.  However, this gives us 30 retry opportunities (1 month) if something is wrong on the first try.
  
  All configs are centralized in `env.local`, easing reproducibility on multiple deployments of PAVICS and following infra-as-code.
  
  User can still perform the renewal manually by calling `certbotwrapper` directly. User is not forced to enable the "scheduler" component but will miss out on the automatic renewal.
  
  Documentation for activating this automatic renewal is in `env.local.example`.
  
  See `vagrant-utils/configure-pavics.sh` for how it's being used for real in a Vagrant box.
  
  Logs (`/var/log/PAVICS/renew_letsencrypt_ssl.log`) when no renewal is necessary, proxy down time less than 1 minute:
  [certbot-renew-no-ops.txt](https://github.com/bird-house/birdhouse-deploy/files/5209376/certbot-renew-no-ops.txt)
  ```
  ==========
  certbotwrapper START_TIME=2020-09-11T01:20:02+0000
  + realpath /vagrant/birdhouse/deployment/certbotwrapper
  + THIS_FILE=/vagrant/birdhouse/deployment/certbotwrapper
  + dirname /vagrant/birdhouse/deployment/certbotwrapper
  + THIS_DIR=/vagrant/birdhouse/deployment
  + pwd
  + SAVED_PWD=/
  + . /vagrant/birdhouse/deployment/../default.env
  + export 'DOCKER_NOTEBOOK_IMAGE=pavics/workflow-tests:200803'
  + export 'FINCH_IMAGE=birdhouse/finch:version-0.5.2'
  + export 'THREDDS_IMAGE=unidata/thredds-docker:4.6.14'
  + export 'JUPYTERHUB_USER_DATA_DIR=/data/jupyterhub_user_data'
  + export 'JUPYTER_DEMO_USER=demo'
  + export 'JUPYTER_DEMO_USER_MEM_LIMIT=2G'
  + export 'JUPYTER_DEMO_USER_CPU_LIMIT=0.5'
  + export 'JUPYTER_LOGIN_BANNER_TOP_SECTION='
  + export 'JUPYTER_LOGIN_BANNER_BOTTOM_SECTION='
  + export 'CANARIE_MONITORING_EXTRA_CONF_DIR=/conf.d'
  + export 'THREDDS_ORGANIZATION=Birdhouse'
  + export 'MAGPIE_DB_NAME=magpiedb'
  + export 'VERIFY_SSL=true'
  + export 'AUTODEPLOY_DEPLOY_KEY_ROOT_DIR=/root/.ssh'
  + export 'AUTODEPLOY_PLATFORM_FREQUENCY=7 5 * * *'
  + export 'AUTODEPLOY_NOTEBOOK_FREQUENCY=@hourly'
  + ENV_LOCAL_FILE=/vagrant/birdhouse/deployment/../env.local
  + set +x
  + CERT_DOMAIN=
  + '[' -z  ]
  + CERT_DOMAIN=lvupavicsmaster.ouranos.ca
  + '[' '!' -z 1 ]
  + cd /vagrant/birdhouse/deployment/..
  + docker stop proxy
  proxy
  + cd /
  + CERTBOT_OPTS=
  + '[' '!' -z 1 ]
  + CERTBOT_OPTS=renew
  + docker run --rm --name certbot -v /etc/letsencrypt:/etc/letsencrypt -v /var/lib/letsencrypt:/var/lib/letsencrypt -v /var/log/letsencrypt:/var/log/letsencrypt -p 443:443 -p 80:80 certbot/certbot:v1.3.0 renew
  Saving debug log to /var/log/letsencrypt/letsencrypt.log
  
  - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
  Processing /etc/letsencrypt/renewal/lvupavicsmaster.ouranos.ca.conf
  - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
  Cert not yet due for renewal
  
  - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
  
  The following certs are not due for renewal yet:
    /etc/letsencrypt/live/lvupavicsmaster.ouranos.ca/fullchain.pem expires on 2020-11-02 (skipped)
  No renewals were attempted.
  - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
  + RC=0
  + '[' '!' -z 1 ]
  + TMP_SSL_CERT=/tmp/tmp_certbotwrapper_ssl_cert.pem
  + CERTPATH=/etc/letsencrypt/live/lvupavicsmaster.ouranos.ca
  + cd /vagrant/birdhouse/deployment/..
  + docker run --rm --name copy_cert -v /etc/letsencrypt:/etc/letsencrypt bash cat /etc/letsencrypt/live/lvupavicsmaster.ouranos.ca/fullchain.pem /etc/letsencrypt/live/lvupavicsmaster.ouranos.ca/privkey.pem
  + diff /home/vagrant/certkey.pem /tmp/tmp_certbotwrapper_ssl_cert.pem
  + rm -v /tmp/tmp_certbotwrapper_ssl_cert.pem
  removed '/tmp/tmp_certbotwrapper_ssl_cert.pem'
  + '[' -z  ]
  + docker start proxy
  proxy
  + cd /
  + set +x
  
  certbotwrapper finished START_TIME=2020-09-11T01:20:02+0000
  certbotwrapper finished   END_TIME=2020-09-11T01:20:21+0000
  ```
  
  Logs when renewal is needed but failed due to firewall, `certbot` adds a random delay so proxy could be down up to 10 mins:
  [certbot-renew-error.txt](https://github.com/bird-house/birdhouse-deploy/files/5209403/certbot-renew-error.txt)
  
  ```
  ==========
  certbotwrapper START_TIME=2020-09-11T13:00:04+0000
  + realpath /home/mourad/PROJECTS/birdhouse-deploy/birdhouse/deployment/certbotwrapper
  + THIS_FILE=/home/mourad/PROJECTS/birdhouse-deploy/birdhouse/deployment/certbotwrapper
  + dirname /home/mourad/PROJECTS/birdhouse-deploy/birdhouse/deployment/certbotwrapper
  + THIS_DIR=/home/mourad/PROJECTS/birdhouse-deploy/birdhouse/deployment
  + pwd
  + SAVED_PWD=/
  + . /home/mourad/PROJECTS/birdhouse-deploy/birdhouse/deployment/../default.env
  + export 'DOCKER_NOTEBOOK_IMAGE=pavics/workflow-tests:200803'
  + export 'FINCH_IMAGE=birdhouse/finch:version-0.5.2'
  + export 'THREDDS_IMAGE=unidata/thredds-docker:4.6.14'
  + export 'JUPYTERHUB_USER_DATA_DIR=/data/jupyterhub_user_data'
  + export 'JUPYTER_DEMO_USER=demo'
  + export 'JUPYTER_DEMO_USER_MEM_LIMIT=2G'
  + export 'JUPYTER_DEMO_USER_CPU_LIMIT=0.5'
  + export 'JUPYTER_LOGIN_BANNER_TOP_SECTION='
  + export 'JUPYTER_LOGIN_BANNER_BOTTOM_SECTION='
  + export 'CANARIE_MONITORING_EXTRA_CONF_DIR=/conf.d'
  + export 'THREDDS_ORGANIZATION=Birdhouse'
  + export 'MAGPIE_DB_NAME=magpiedb'
  + export 'VERIFY_SSL=true'
  + export 'AUTODEPLOY_DEPLOY_KEY_ROOT_DIR=/root/.ssh'
  + export 'AUTODEPLOY_PLATFORM_FREQUENCY=7 5 * * *'
  + export 'AUTODEPLOY_NOTEBOOK_FREQUENCY=@hourly'
  + ENV_LOCAL_FILE=/home/mourad/PROJECTS/birdhouse-deploy/birdhouse/deployment/../env.local
  + set +x
  + CERT_DOMAIN=
  + '[' -z  ]
  + CERT_DOMAIN=medus.ouranos.ca
  + '[' '!' -z 1 ]
  + cd /home/mourad/PROJECTS/birdhouse-deploy/birdhouse/deployment/..
  + docker stop proxy
  proxy
  + cd /
  + CERTBOT_OPTS=
  + '[' '!' -z 1 ]
  + CERTBOT_OPTS=renew
  + docker run --rm --name certbot -v /etc/letsencrypt:/etc/letsencrypt -v /var/lib/letsencrypt:/var/lib/letsencrypt -v /var/log/letsencrypt:/var/log/letsencrypt -p 443:443 -p 80:80 certbot/certbot:v1.3.0 renew
  Saving debug log to /var/log/letsencrypt/letsencrypt.log
  
  - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
  Processing /etc/letsencrypt/renewal/medus.ouranos.ca.conf
  - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
  Cert is due for renewal, auto-renewing...
  Non-interactive renewal: random delay of 10.77459918236335 seconds
  Plugins selected: Authenticator standalone, Installer None
  Renewing an existing certificate
  Performing the following challenges:
  http-01 challenge for medus.ouranos.ca
  Waiting for verification...
  Challenge failed for domain medus.ouranos.ca
  http-01 challenge for medus.ouranos.ca
  Cleaning up challenges
  Attempting to renew cert (medus.ouranos.ca) from /etc/letsencrypt/renewal/medus.ouranos.ca.conf produced an unexpected error: Some challenges have failed.. Skipping.
  All renewal attempts failed. The following certs could not be renewed:
    /etc/letsencrypt/live/medus.ouranos.ca/fullchain.pem (failure)
  
  - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
  
  All renewal attempts failed. The following certs could not be renewed:
    /etc/letsencrypt/live/medus.ouranos.ca/fullchain.pem (failure)
  - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
  1 renew failure(s), 0 parse failure(s)
  IMPORTANT NOTES:
   - The following errors were reported by the server:
  
     Domain: medus.ouranos.ca
     Type:   connection
     Detail: Fetching
     http://medus.ouranos.ca/.well-known/acme-challenge/F-_TzoOMcgoo5WC9FQvi_QdKuoqdsrQFa7MR2bEdnJE:
     Timeout during connect (likely firewall problem)
  
     To fix these errors, please make sure that your domain name was
     entered correctly and the DNS A/AAAA record(s) for that domain
     contain(s) the right IP address. Additionally, please check that
     your computer has a publicly routable IP address and that no
     firewalls are preventing the server from communicating with the
     client. If you're using the webroot plugin, you should also verify
     that you are serving files from the webroot path you provided.
  + RC=1
  + '[' '!' -z 1 ]
  + TMP_SSL_CERT=/tmp/tmp_certbotwrapper_ssl_cert.pem
  + CERTPATH=/etc/letsencrypt/live/medus.ouranos.ca
  + cd /home/mourad/PROJECTS/birdhouse-deploy/birdhouse/deployment/..
  + docker run --rm --name copy_cert -v /etc/letsencrypt:/etc/letsencrypt bash cat /etc/letsencrypt/live/medus.ouranos.ca/fullchain.pem /etc/letsencrypt/live/medus.ouranos.ca/privkey.pem
  + diff /etc/letsencrypt/live/medus.ouranos.ca/certkey.pem /tmp/tmp_certbotwrapper_ssl_cert.pem
  + rm -v /tmp/tmp_certbotwrapper_ssl_cert.pem
  removed '/tmp/tmp_certbotwrapper_ssl_cert.pem'
  + '[' -z  ]
  + docker start proxy
  proxy
  + cd /
  + set +x
  
  certbotwrapper finished START_TIME=2020-09-11T13:00:04+0000
  certbotwrapper finished   END_TIME=2020-09-11T13:00:49+0000
  ```
  
  Logs when renewal is successful, again proxy could be down up to 10 mins due to random delay by `certbot` client:
  [certbot-renew-success-in-2-run-after-file-copy-fix.txt](https://github.com/bird-house/birdhouse-deploy/files/5209924/certbot-renew-success-in-2-run-after-file-copy-fix.txt)
  
  ```
  ==========
  certbotwrapper START_TIME=2020-09-11T13:10:04+0000
  + realpath /home/mourad/PROJECTS/birdhouse-deploy/birdhouse/deployment/certbotwrapper
  + THIS_FILE=/home/mourad/PROJECTS/birdhouse-deploy/birdhouse/deployment/certbotwrapper
  + dirname /home/mourad/PROJECTS/birdhouse-deploy/birdhouse/deployment/certbotwrapper
  + THIS_DIR=/home/mourad/PROJECTS/birdhouse-deploy/birdhouse/deployment
  + pwd
  + SAVED_PWD=/
  + . /home/mourad/PROJECTS/birdhouse-deploy/birdhouse/deployment/../default.env
  + export 'DOCKER_NOTEBOOK_IMAGE=pavics/workflow-tests:200803'
  + export 'FINCH_IMAGE=birdhouse/finch:version-0.5.2'
  + export 'THREDDS_IMAGE=unidata/thredds-docker:4.6.14'
  + export 'JUPYTERHUB_USER_DATA_DIR=/data/jupyterhub_user_data'
  + export 'JUPYTER_DEMO_USER=demo'
  + export 'JUPYTER_DEMO_USER_MEM_LIMIT=2G'
  + export 'JUPYTER_DEMO_USER_CPU_LIMIT=0.5'
  + export 'JUPYTER_LOGIN_BANNER_TOP_SECTION='
  + export 'JUPYTER_LOGIN_BANNER_BOTTOM_SECTION='
  + export 'CANARIE_MONITORING_EXTRA_CONF_DIR=/conf.d'
  + export 'THREDDS_ORGANIZATION=Birdhouse'
  + export 'MAGPIE_DB_NAME=magpiedb'
  + export 'VERIFY_SSL=true'
  + export 'AUTODEPLOY_DEPLOY_KEY_ROOT_DIR=/root/.ssh'
  + export 'AUTODEPLOY_PLATFORM_FREQUENCY=7 5 * * *'
  + export 'AUTODEPLOY_NOTEBOOK_FREQUENCY=@hourly'
  + ENV_LOCAL_FILE=/home/mourad/PROJECTS/birdhouse-deploy/birdhouse/deployment/../env.local
  + set +x
  + CERT_DOMAIN=
  + '[' -z  ]
  + CERT_DOMAIN=medus.ouranos.ca
  + '[' '!' -z 1 ]
  + cd /home/mourad/PROJECTS/birdhouse-deploy/birdhouse/deployment/..
  + docker stop proxy
  proxy
  + cd /
  + CERTBOT_OPTS=
  + '[' '!' -z 1 ]
  + CERTBOT_OPTS=renew
  + docker run --rm --name certbot -v /etc/letsencrypt:/etc/letsencrypt -v /var/lib/letsencrypt:/var/lib/letsencrypt -v /var/log/letsencrypt:/var/log/letsencrypt -p 443:443 -p 80:80 certbot/certbot:v1.3.0 renew
  Saving debug log to /var/log/letsencrypt/letsencrypt.log
  
  - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
  Processing /etc/letsencrypt/renewal/medus.ouranos.ca.conf
  - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
  Cert is due for renewal, auto-renewing...
  Non-interactive renewal: random delay of 459.45712705256506 seconds
  Plugins selected: Authenticator standalone, Installer None
  Renewing an existing certificate
  Performing the following challenges:
  http-01 challenge for medus.ouranos.ca
  Waiting for verification...
  Cleaning up challenges
  
  - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
  new certificate deployed without reload, fullchain is
  /etc/letsencrypt/live/medus.ouranos.ca/fullchain.pem
  - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
  
  - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
  
  Congratulations, all renewals succeeded. The following certs have been renewed:
    /etc/letsencrypt/live/medus.ouranos.ca/fullchain.pem (success)
  - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
  + RC=0
  + '[' '!' -z 1 ]
  + TMP_SSL_CERT=/tmp/tmp_certbotwrapper_ssl_cert.pem
  + CERTPATH=/etc/letsencrypt/live/medus.ouranos.ca
  + cd /home/mourad/PROJECTS/birdhouse-deploy/birdhouse/deployment/..
  + docker run --rm --name copy_cert -v /etc/letsencrypt:/etc/letsencrypt bash cat /etc/letsencrypt/live/medus.ouranos.ca/fullchain.pem /etc/letsencrypt/live/medus.ouranos.ca/privkey.pem
  + diff /etc/letsencrypt/live/medus.ouranos.ca/certkey.pem /tmp/tmp_certbotwrapper_ssl_cert.pem
  --- /etc/letsencrypt/live/medus.ouranos.ca/certkey.pem
  +++ /tmp/tmp_certbotwrapper_ssl_cert.pem
  @@ -1,33 +1,33 @@
   -----BEGIN CERTIFICATE-----
  
  REMOVED for Privacy.
  
   -----END PRIVATE KEY-----
  + '[' 0 -eq 0 ]
  + cp -v /tmp/tmp_certbotwrapper_ssl_cert.pem /etc/letsencrypt/live/medus.ouranos.ca/certkey.pem
  cp: can't create '/etc/letsencrypt/live/medus.ouranos.ca/certkey.pem': File exists
  + rm -v /tmp/tmp_certbotwrapper_ssl_cert.pem
  removed '/tmp/tmp_certbotwrapper_ssl_cert.pem'
  + '[' -z  ]
  + docker start proxy
  proxy
  + cd /
  + set +x
  
  certbotwrapper finished START_TIME=2020-09-11T13:10:04+0000
  certbotwrapper finished   END_TIME=2020-09-11T13:18:10+0000
  ==========
  certbotwrapper START_TIME=2020-09-11T15:00:06+0000
  + realpath /home/mourad/PROJECTS/birdhouse-deploy/birdhouse/deployment/certbotwrapper
  + THIS_FILE=/home/mourad/PROJECTS/birdhouse-deploy/birdhouse/deployment/certbotwrapper
  + dirname /home/mourad/PROJECTS/birdhouse-deploy/birdhouse/deployment/certbotwrapper
  + THIS_DIR=/home/mourad/PROJECTS/birdhouse-deploy/birdhouse/deployment
  + pwd
  + SAVED_PWD=/
  + . /home/mourad/PROJECTS/birdhouse-deploy/birdhouse/deployment/../default.env
  + export 'DOCKER_NOTEBOOK_IMAGE=pavics/workflow-tests:200803'
  + export 'FINCH_IMAGE=birdhouse/finch:version-0.5.2'
  + export 'THREDDS_IMAGE=unidata/thredds-docker:4.6.14'
  + export 'JUPYTERHUB_USER_DATA_DIR=/data/jupyterhub_user_data'
  + export 'JUPYTER_DEMO_USER=demo'
  + export 'JUPYTER_DEMO_USER_MEM_LIMIT=2G'
  + export 'JUPYTER_DEMO_USER_CPU_LIMIT=0.5'
  + export 'JUPYTER_LOGIN_BANNER_TOP_SECTION='
  + export 'JUPYTER_LOGIN_BANNER_BOTTOM_SECTION='
  + export 'CANARIE_MONITORING_EXTRA_CONF_DIR=/conf.d'
  + export 'THREDDS_ORGANIZATION=Birdhouse'
  + export 'MAGPIE_DB_NAME=magpiedb'
  + export 'VERIFY_SSL=true'
  + export 'AUTODEPLOY_DEPLOY_KEY_ROOT_DIR=/root/.ssh'
  + export 'AUTODEPLOY_PLATFORM_FREQUENCY=7 5 * * *'
  + export 'AUTODEPLOY_NOTEBOOK_FREQUENCY=@hourly'
  + ENV_LOCAL_FILE=/home/mourad/PROJECTS/birdhouse-deploy/birdhouse/deployment/../env.local
  + set +x
  + CERT_DOMAIN=
  + '[' -z  ]
  + CERT_DOMAIN=medus.ouranos.ca
  + '[' '!' -z 1 ]
  + cd /home/mourad/PROJECTS/birdhouse-deploy/birdhouse/deployment/..
  + docker stop proxy
  proxy
  + cd /
  + CERTBOT_OPTS=
  + '[' '!' -z 1 ]
  + CERTBOT_OPTS=renew
  + docker run --rm --name certbot -v /etc/letsencrypt:/etc/letsencrypt -v /var/lib/letsencrypt:/var/lib/letsencrypt -v /var/log/letsencrypt:/var/log/letsencrypt -p 443:443 -p 80:80 certbot/certbot:v1.3.0 renew
  Saving debug log to /var/log/letsencrypt/letsencrypt.log
  
  - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
  Processing /etc/letsencrypt/renewal/medus.ouranos.ca.conf
  - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
  Cert not yet due for renewal
  
  - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
  
  The following certs are not due for renewal yet:
    /etc/letsencrypt/live/medus.ouranos.ca/fullchain.pem expires on 2020-12-10 (skipped)
  No renewals were attempted.
  - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
  + RC=0
  + '[' '!' -z 1 ]
  + TMP_SSL_CERT=/tmp/tmp_certbotwrapper_ssl_cert.pem
  + CERTPATH=/etc/letsencrypt/live/medus.ouranos.ca
  + cd /home/mourad/PROJECTS/birdhouse-deploy/birdhouse/deployment/..
  + docker run --rm --name copy_cert -v /etc/letsencrypt:/etc/letsencrypt bash cat /etc/letsencrypt/live/medus.ouranos.ca/fullchain.pem /etc/letsencrypt/live/medus.ouranos.ca/privkey.pem
  + diff /etc/letsencrypt/live/medus.ouranos.ca/certkey.pem /tmp/tmp_certbotwrapper_ssl_cert.pem
  --- /etc/letsencrypt/live/medus.ouranos.ca/certkey.pem
  +++ /tmp/tmp_certbotwrapper_ssl_cert.pem
  @@ -1,33 +1,33 @@
   -----BEGIN CERTIFICATE-----
  
  REMOVED for Privacy.
  
   -----END PRIVATE KEY-----
  + '[' 0 -eq 0 ]
  + cp -v /tmp/tmp_certbotwrapper_ssl_cert.pem /etc/letsencrypt/live/medus.ouranos.ca/certkey.pem
  '/tmp/tmp_certbotwrapper_ssl_cert.pem' -> '/etc/letsencrypt/live/medus.ouranos.ca/certkey.pem'
  + rm -v /tmp/tmp_certbotwrapper_ssl_cert.pem
  removed '/tmp/tmp_certbotwrapper_ssl_cert.pem'
  + '[' -z  ]
  + docker start proxy
  proxy
  + cd /
  + set +x
  
  certbotwrapper finished START_TIME=2020-09-11T15:00:06+0000
  certbotwrapper finished   END_TIME=2020-09-11T15:00:31+0000
  ```

`1.11.1 <https://github.com/bird-house/birdhouse-deploy/tree/1.11.1>`_ (2020-09-15)
========================================================================================================================

- jupyter: new updated image with new handcalcs package
  
  Matching PR to deploy the new jupyter image to PAVICS.
  
  See PR https://github.com/Ouranosinc/PAVICS-e2e-workflow-tests/pull/50
  (commit https://github.com/Ouranosinc/PAVICS-e2e-workflow-tests/commit/02333bfa11931f4a0b7c9607b88904bd063bed70)
  that built the new image with the detailed change vs the previous image.
  
  Add handcalcs https://github.com/connorferster/handcalcs/ and unpin hvplot
  since pinning did not solve violin plot issue, see this comment
  https://github.com/bird-house/birdhouse-deploy/pull/63#issuecomment-668270608
  
  Successful Jenkins build
  http://jenkins.ouranos.ca/job/PAVICS-e2e-workflow-tests/job/periodic-rebuild-and-add-handcalcs/1/console
  
  Noticeable changes:
  ```diff
  >     - handcalcs==0.8.1
  
  <     - xclim==0.18.0
  >     - xclim==0.19.0
  
  <   - hvplot=0.5.2=py_0
  >   - hvplot=0.6.0=pyh9f0ad1d_0
  
  <   - dask=2.22.0=py_0
  >   - dask=2.26.0=py_0
  
  <   - bokeh=2.1.1=py37hc8dfbb8_0
  >   - bokeh=2.2.1=py37hc8dfbb8_0
  
  <   - numba=0.50.1=py37h0da4684_1
  >   - numba=0.51.2=py37h9fdb41a_0
  ```

`1.11.0 <https://github.com/bird-house/birdhouse-deploy/tree/1.11.0>`_ (2020-08-25)
========================================================================================================================

- Improved plugable component architecture.
  
  Before this PR, components needing default values, needing template variable substitution, needing to execute commands pre and post `docker-compose up` are hardcoding their needs directly to the "core" system, basically "leaking" their requirements out even when they are not activated (fixes https://github.com/bird-house/birdhouse-deploy/issues/62).
  
  This PR provides true plugable architecture for the components so they can provide all their needs without having to modify the code of the "core" system.
  
  All the components (monitoring, generic_bird, emu, testthredds) are modified to leverage the new plugable architecture, with additional customizations given it is cleaner/easier to have default configuration values.
  
  Given this PR both changes the architecture and modify many components at the same time, it is best to read each commit separately to easier understand which code change belongs to which "goal".
  
  Deployed here https://lvupavicsmaster.ouranos.ca with all the impacted components activated to test the change:
  * Canarie: https://lvupavicsmaster.ouranos.ca/canarie/node/service/status
  * Generic bird (using Finch): https://lvupavicsmaster.ouranos.ca/twitcher/ows/proxy/generic_bird?service=WPS&version=1.0.0&request=GetCapabilities
  * Emu: https://lvupavicsmaster.ouranos.ca/twitcher/ows/proxy/emu?service=WPS&version=1.0.0&request=GetCapabilities
  * Test Thredds: https://lvupavicsmaster.ouranos.ca/testthredds/catalog.html
  * Prometheus: http://lvupavicsmaster.ouranos.ca:9090/alerts
  * AlertManager: http://lvupavicsmaster.ouranos.ca:9093/
  * Grafana dashboard: http://lvupavicsmaster.ouranos.ca:3001/d/pf6xQMWGz/docker-and-system-monitoring?orgId=1&refresh=5m

`1.10.4 <https://github.com/bird-house/birdhouse-deploy/tree/1.10.4>`_ (2020-08-05)
========================================================================================================================

- jupyter: new update image with hvplot pinned to older version for violin plot
  
  See PR https://github.com/Ouranosinc/PAVICS-e2e-workflow-tests/pull/48 (commit https://github.com/Ouranosinc/PAVICS-e2e-workflow-tests/commit/4ad6ba6fa2a4ecf6d5d78e0602b39202307bcb76) for more detailed info.
  
  Deployed to Medus for testing (as regular PAVICS image, not the devel image).  @aulemahal reported back that violin plot still do not work even with the old hvplot pinned in this image.
  
  I'll release this image as-is since violin plot is also not working in the previous image that had hvplot 0.6.0 so no new regression there.  Will unpin hvplot on next image build because pinning it did not fix violin plot (probably interference from other newer packages in this build).
  
  Noticeable changes:
  ```diff
  <   - hvplot=0.6.0=pyh9f0ad1d_0
  >   - hvplot=0.5.2=py_0
  
  <   - dask=2.20.0=py_0
  >   - dask=2.22.0=py_0
  
  <   - geopandas=0.8.0=py_1
  >   - geopandas=0.8.1=py_0
  
  <   - pandas=1.0.5=py37h0da4684_0
  >   - pandas=1.1.0=py37h3340039_0
  
  <   - matplotlib=3.2.2=1
  >   - matplotlib=3.3.0=1
  
  <   - numpy=1.18.5=py37h8960a57_0
  >   - numpy=1.19.1=py37h8960a57_0
  
  <   - cryptography=2.9.2=py37hb09aad4_0
  >   - cryptography=3.0=py37hb09aad4_0
  
  <   - python=3.7.6=h8356626_5_cpython
  >   - python=3.7.8=h6f2ec95_1_cpython
  
  <   - nbval=0.9.5=py_0
  >   - nbval=0.9.6=pyh9f0ad1d_0
  
  <   - pytest=5.4.3=py37hc8dfbb8_0
  >   - pytest=6.0.1=py37hc8dfbb8_0
  ```

`1.10.3 <https://github.com/bird-house/birdhouse-deploy/tree/1.10.3>`_ (2020-07-21)
========================================================================================================================

- proxy: increase timeout for reading a response from the proxied server
  
  Fixes https://github.com/Ouranosinc/raven/issues/286
  
  "there seems to be a problem with the size of the ncml and the timeout
  if I use more than 10-12 years as the historical data. I get a :
  "Netcdf: DAP failure" error if I use too many years."
  
  ```
  ________________________________________________________ TestBiasCorrect.test_bias_correction ________________________________________________________
  Traceback (most recent call last):
    File "/zstore/repos/raven/tests/test_bias_correction.py", line 20, in test_bias_correction
      ds = (xr.open_dataset(hist_data).sel(lat=slice(lat + 1, lat - 1),lon=slice(lon - 1, lon + 1), time=slice(dt.datetime(1991,1,1), dt.datetime(2010,12,31))).mean(dim={"lat", "lon"}, keep_attrs=True))
    File "/home/lvu/.conda/envs/raven/lib/python3.7/site-packages/xarray/core/common.py", line 84, in wrapped_func
      func, dim, skipna=skipna, numeric_only=numeric_only, **kwargs
    File "/home/lvu/.conda/envs/raven/lib/python3.7/site-packages/xarray/core/dataset.py", line 4313, in reduce
      **kwargs,
    File "/home/lvu/.conda/envs/raven/lib/python3.7/site-packages/xarray/core/variable.py", line 1586, in reduce
      input_data = self.data if allow_lazy else self.values
    File "/home/lvu/.conda/envs/raven/lib/python3.7/site-packages/xarray/core/variable.py", line 349, in data
      return self.values
    File "/home/lvu/.conda/envs/raven/lib/python3.7/site-packages/xarray/core/variable.py", line 457, in values
      return _as_array_or_item(self._data)
    File "/home/lvu/.conda/envs/raven/lib/python3.7/site-packages/xarray/core/variable.py", line 260, in _as_array_or_item
      data = np.asarray(data)
    File "/home/lvu/.conda/envs/raven/lib/python3.7/site-packages/numpy/core/_asarray.py", line 83, in asarray
      return array(a, dtype, copy=False, order=order)
    File "/home/lvu/.conda/envs/raven/lib/python3.7/site-packages/xarray/core/indexing.py", line 677, in __array__
      self._ensure_cached()
    File "/home/lvu/.conda/envs/raven/lib/python3.7/site-packages/xarray/core/indexing.py", line 674, in _ensure_cached
      self.array = NumpyIndexingAdapter(np.asarray(self.array))
    File "/home/lvu/.conda/envs/raven/lib/python3.7/site-packages/numpy/core/_asarray.py", line 83, in asarray
      return array(a, dtype, copy=False, order=order)
    File "/home/lvu/.conda/envs/raven/lib/python3.7/site-packages/xarray/core/indexing.py", line 653, in __array__
      return np.asarray(self.array, dtype=dtype)
    File "/home/lvu/.conda/envs/raven/lib/python3.7/site-packages/numpy/core/_asarray.py", line 83, in asarray
      return array(a, dtype, copy=False, order=order)
    File "/home/lvu/.conda/envs/raven/lib/python3.7/site-packages/xarray/core/indexing.py", line 557, in __array__
      return np.asarray(array[self.key], dtype=None)
    File "/home/lvu/.conda/envs/raven/lib/python3.7/site-packages/xarray/backends/netCDF4_.py", line 73, in __getitem__
      key, self.shape, indexing.IndexingSupport.OUTER, self._getitem
    File "/home/lvu/.conda/envs/raven/lib/python3.7/site-packages/xarray/core/indexing.py", line 837, in explicit_indexing_adapter
      result = raw_indexing_method(raw_key.tuple)
    File "/home/lvu/.conda/envs/raven/lib/python3.7/site-packages/xarray/backends/netCDF4_.py", line 85, in _getitem
      array = getitem(original_array, key)
    File "/home/lvu/.conda/envs/raven/lib/python3.7/site-packages/xarray/backends/common.py", line 54, in robust_getitem
      return array[key]
    File "netCDF4/_netCDF4.pyx", line 4408, in netCDF4._netCDF4.Variable.__getitem__
    File "netCDF4/_netCDF4.pyx", line 5352, in netCDF4._netCDF4.Variable._get
    File "netCDF4/_netCDF4.pyx", line 1887, in netCDF4._netCDF4._ensure_nc_success
  RuntimeError: NetCDF: DAP failure
  ```

`1.10.2 <https://github.com/bird-house/birdhouse-deploy/tree/1.10.2>`_ (2020-07-18)
========================================================================================================================

- jupyter: new build and add nc-time-axis
  
  Corresponding change to deploy the new Jupyter env to PAVICS.
  
  Noticeable changes:
  ```diff
  <   - dask=2.17.2=py_0
  >   - dask=2.20.0=py_0
  
  >   - nc-time-axis=1.2.0=py_1
  
  <   - xarray=0.15.1=py_0
  >   - xarray=0.16.0=py_0
  
  <     - xclim==0.17.0
  >     - xclim==0.18.0
  ```
  
  See PR https://github.com/Ouranosinc/PAVICS-e2e-workflow-tests/pull/47
  (commit
  https://github.com/Ouranosinc/PAVICS-e2e-workflow-tests/commit/4e03a674930f0974e13724940eee7a608c2158c0)
  for more info.

`1.10.1 <https://github.com/bird-house/birdhouse-deploy/tree/1.10.1>`_ (2020-07-11)
========================================================================================================================

- Monitoring: add alert rules and alert handling (deduplicate, group, route, silence, inhibit).
  
  This is a follow up to the previous PR https://github.com/bird-house/birdhouse-deploy/pull/56 that added the monitoring itself.
  
  Added cAdvisor and Node-exporter collection of alert rules found here https://awesome-prometheus-alerts.grep.to/rules with a few fixing because of errors in the rules and tweaking to reduce false positive alarms (see list of commits).  Great collection of sample of ready-made rules to hit the ground running and learn PromML query language on the way.
  
  ![2020-07-08-090953_474x1490_scrot](https://user-images.githubusercontent.com/11966697/86926000-8b086c80-c0ff-11ea-92d0-6f5ccfe2b8e1.png)
  
  Added Alertmanager to handle the alerts (deduplicate, group, route, silence, inhibit).  Currently the only notification route configured is email but Alertmanager is able to route alerts to Slack and any generic services accepting webhooks.
  
  ![2020-07-08-091150_1099x669_scrot](https://user-images.githubusercontent.com/11966697/86926213-cd31ae00-c0ff-11ea-8b2a-d33803ad3d5d.png)
  
  ![2020-07-08-091302_1102x1122_scrot](https://user-images.githubusercontent.com/11966697/86926276-dc186080-c0ff-11ea-9377-bda03b69640e.png)
  
  This is an initial attempt at alerting.  There are several ways to tweak the system without changing the code:
  
  * To add more Prometheus alert rules, volume-mount more *.rules files to the prometheus container.
  * To disable existing Prometheus alert rules, add more Alertmanager inhibition rules using `ALERTMANAGER_EXTRA_INHIBITION` via `env.local` file.
  * Other possible Alertmanager configs via `env.local`: `ALERTMANAGER_EXTRA_GLOBAL, ALERTMANAGER_EXTRA_ROUTES, ALERTMANAGER_EXTRA_RECEIVERS`.
  
  What more could be done after this initial attempt:
  
  * Possibly add more graphs to Grafana dashboard since we have more alerts on metrics that we do not have matching Grafana graph. Graphs are useful for historical trends and correlation with other metrics, so not required if we do not need trends and correlation.
  
  * Only basic metrics are being collected currently.  We could collect more useful metrics like SMART status and alert when a disk is failing.
  
  * The autodeploy mechanism can hook into this monitoring system to report pass/fail status and execution duration, with alerting for problems.  Then we can also correlate any CPU, memory, disk I/O spike, when the autodeploy runs and have a trace of previous autodeploy executions.
  
  I had to test these alerts directly in prod to tweak for less false positive alert and to debug not working rules to ensure they work on prod so these changes are already in prod !   This also test the SMTP server on the network.
  
  See rules on Prometheus side: http://pavics.ouranos.ca:9090/rules, http://medus.ouranos.ca:9090/rules
  
  Manage alerts on Alertmanager side: http://pavics.ouranos.ca:9093/#/alerts, http://medus.ouranos.ca:9093/#/alerts
  
  Part of issue https://github.com/bird-house/birdhouse-deploy/issues/12

`1.10.0 <https://github.com/bird-house/birdhouse-deploy/tree/1.10.0>`_ (2020-07-02)
========================================================================================================================

- Monitoring for host and each docker container.
  
  ![Screenshot_2020-06-19 Docker and system monitoring - Grafana](https://user-images.githubusercontent.com/11966697/85206384-c7f6f580-b2ef-11ea-848d-46490eb95886.png)
  
  For host, using Node-exporter to collect metrics:
  * uptime
  * number of container
  * used disk space
  * used memory, available memory, used swap memory
  * load
  * cpu usage
  * in and out network traffic 
  * disk I/O
  
  For each container, using cAdvisor to collect metrics:
  * in and out network traffic
  * cpu usage
  * memory and swap memory usage
  * disk usage
  
  Useful visualisation features:
  * zoom in one graph and all other graph update to match the same "time range" so we can correlate event
  * view each graph independently for more details
  * mouse over each data point will show value at that moment
  
  Prometheus is used as the time series DB and Grafana is used as the visualization dashboard.
  
  Node-exporter, cAdvisor and Prometheus are exposed so another Prometheus on the network can also scrape those same metrics and perform other analysis if required.
  
  The whole monitoring stack is a separate component so user is not forced to enable it if there is already another monitoring system in place.  Enabling this monitoring stack is done via `env.local` file, like all other components.
  
  The Grafana dashboard is taken from https://grafana.com/grafana/dashboards/893 with many fixes (see commits) since most of the metric names have changed over time.  Still it was much quicker to hit the ground running than learning the Prometheus query language and Grafana visualization options from scratch.  Not counting there are lots of metrics exposed, had to filter out which one are relevant to graph.  So starting from a broken dashboard was still a big win.  Grafana has a big collection of existing but probably un-maintained dashboards we can leverage.
  
  So this is a first draft for monitoring.  Many things I am not sure or will need tweaking or is missing:
  * Probably have to add more metrics or remove some that might be irrelevant, with time we will see.
  * Probably will have to tweak the scrape interval and the retention time, to keep the disk storage requirement reasonable, again we'll see with time.
  * Missing alerting.  With all the pretty graph, we are not going to look at them all day, we need some kind of alerting mechanism.
  
  Test system: http://lvupavicsmaster.ouranos.ca:3001/d/pf6xQMWGz/docker-and-system-monitoring?orgId=1&refresh=5m, user: admin, passwd: the default passwd
  
  Also tested on Medus: http://medus.ouranos.ca:3001/d/pf6xQMWGz/docker-and-system-monitoring?orgId=1&refresh=5m (on Medus had to perform full yum update to get new kernel and new docker engine for cAdvisor to work properly).
  
  Part of issue https://github.com/bird-house/birdhouse-deploy/issues/12

`1.9.6 <https://github.com/bird-house/birdhouse-deploy/tree/1.9.6>`_ (2020-06-15)
========================================================================================================================

- flyingpigeon: update to version 1.6
  
  Deploy the new Flyingpigeon 1.6 on PAVICS.
  
  Has been deployed to Medus test environment.
  
  flyingpigeon changelog from release commit
  https://github.com/bird-house/flyingpigeon/commit/a6f54ed0c20919485c2420295729e30f914cfa15
  (PR https://github.com/bird-house/flyingpigeon/pull/332)
  
  1.6 (2020-06-10)
  ================
  * remove eggshell dependency
  * notebooks are part of the test suite
  * improved plot processes
  * remove mosaic option for subset processes
  * polygon subset processes files separately instead of an entire data-set at once
  * multiple outputs listed in Metalink output
  * update pywps to 4.2.3
  * use cruft to keep up-to-date with the cookie-cutter template

`1.9.5 <https://github.com/bird-house/birdhouse-deploy/tree/1.9.5>`_ (2020-06-12)
========================================================================================================================

- jupyter: new image for additional plugins
  
  Matching PR to deploy the new Jupyter image to PAVICS.
  
  Added:
  
  * https://github.com/hadim/jupyter-archive
    Download entire folder as archive.
  
  * https://blog.jupyter.org/a-visual-debugger-for-jupyter-914e61716559
  
  * https://github.com/plotly/jupyter-dash
    Develop Plotly Dash apps interactively from within Jupyter environments.
  
  Noticeable changes:
  ```diff
  >   - jupyter-archive=0.6.2=py_0
  >   - jupyter-dash=0.2.1.post1=py_0
  
  <   - owslib=0.19.2=py_1
  >   - owslib=0.20.0=py_0
  
  >   - xeus-python=0.7.1=py37h99015e2_1
  ```
  
  See PR https://github.com/Ouranosinc/PAVICS-e2e-workflow-tests/pull/46 (commit
  https://github.com/Ouranosinc/PAVICS-e2e-workflow-tests/commit/441edde3b381eff7ce82e5a171323b31196553be)
  for more info.

`1.9.4 <https://github.com/bird-house/birdhouse-deploy/tree/1.9.4>`_ (2020-06-03)
========================================================================================================================

- jupyter: updated build and fix for pyviz jupyterlab extension
  
  @tlogan2000 matching PR to actually deploy the new Jupyter env to PAVICS.
  
  Noticeable changes:
  
  ```diff
  <   - dask=2.15.0=py_0
  >   - dask=2.17.2=py_0
  
  <     - xclim==0.16.0
  >     - xclim==0.17.0
  ```
  
  See PR https://github.com/Ouranosinc/PAVICS-e2e-workflow-tests/pull/45 (commit
  https://github.com/Ouranosinc/PAVICS-e2e-workflow-tests/commit/a93f3b50cc6d108638d232fe9465b2f060e21314)
  for more info.

`1.9.3 <https://github.com/bird-house/birdhouse-deploy/tree/1.9.3>`_ (2020-05-07)
========================================================================================================================

- jupyter: update to pavics/workflow-tests:200507
  
  Raven PR https://github.com/Ouranosinc/raven/pull/266 (commit
  https://github.com/Ouranosinc/raven/commit/0763bf52abec1bc0a70927de3a2dc2cc1cf77ec3)
  removed salem dependency and replaced with rioxarray.
  
  Also add packages for the
  [`custom_climate_portraits`](https://github.com/Ouranosinc/PAVICS-e2e-workflow-tests/tree/custom_climate_portraits)
  branch  (PR
  https://github.com/Ouranosinc/PAVICS-e2e-workflow-tests/pull/35).
  
  Noticeable changes:
  ```diff
    # conda release of bokeh seems to trail behind pypi
  >   - bokeh=2.0.1=py37hc8dfbb8_0
  <     - bokeh==2.0.2
  >   - jupyter_bokeh=2.0.1=py_0
  
    # should already exist, not sure why conda env export report this as new
  >   - dask=2.15.0=py_0
  
    # unpinned since salem is removed
  <   - pandas=0.25.3=py37hb3f55d8_0
  >   - pandas=1.0.3=py37h0da4684_1
  
  <     - salem==0.2.4
  >   - rioxarray=0.0.26=py_0
  
    # packages for custom_climate_portraits branch
  >   - geoviews=1.8.1=py_0
  >   - h5netcdf=0.8.0=py_0
  >   - holoviews=1.13.2=pyh9f0ad1d_0
  >   - panel=0.9.5=py_1
  >   - hvplot=0.5.2=py_0
  >   - pscript=0.7.3=py_0
  >   - siphon=0.8.0=py37_1002
  >     - ipython-blocking==0.2.1
  ```
  
  See PR https://github.com/Ouranosinc/PAVICS-e2e-workflow-tests/pull/44
  (commit
  https://github.com/Ouranosinc/PAVICS-e2e-workflow-tests/commit/bb81982e3fd92bff437eddc5d4ae28202b3ef07c)
  for more info.

`1.9.2 <https://github.com/bird-house/birdhouse-deploy/tree/1.9.2>`_ (2020-04-29)
========================================================================================================================

- 
  jupyter: update to pavics/workflow-tests:200427 image
  
  See PR https://github.com/Ouranosinc/PAVICS-e2e-workflow-tests/pull/43 (commit https://github.com/Ouranosinc/PAVICS-e2e-workflow-tests/commit/446b2f1ba3342106e3ad3d2dfe16aece7365c492) for more info.
  
  Noticeable changes:
  ```diff
  <   - geopandas=0.6.2=py_0
  >     - geopandas==0.7.0
  
  <   - xarray=0.15.0=py_0
  >   - xarray=0.15.1=py_0
  
  <   - owslib=0.19.1=py_0
  >   - owslib=0.19.2=py_1
  
  <   - dask-core=2.12.0=py_0
  >   - dask-core=2.15.0=py_0
  
  <     - distributed==2.12.0
  >     - distributed==2.15.0
  
  <     - xclim==0.14.0
  >     - xclim==0.16.0
  ```

`1.9.1 <https://github.com/bird-house/birdhouse-deploy/tree/1.9.1>`_ (2020-04-24)
========================================================================================================================

- 
  Fix notebook autodeploy wipe already deployed notebook when GitHub down.
  
  Fixes https://github.com/bird-house/birdhouse-deploy/issues/43
  
  Fail early with any unexpected error to not wipe already deployed notebooks.
  
  Check source dir not empty before wiping dest dir containing already deployed notebooks.
  
  Reduce cleaning verbosity for more concise logging.
  
  To fix this error found in production logs when Github is down today:
  ```
  notebookdeploy START_TIME=2020-04-23T10:01:01-0400
  ++ mktemp -d -t notebookdeploy.XXXXXXXXXXXX
  + TMPDIR=/tmp/notebookdeploy.ICk70Vto2LaE
  + cd /tmp/notebookdeploy.ICk70Vto2LaE
  + mkdir tutorial-notebooks
  + cd tutorial-notebooks
  + wget --quiet https://raw.githubusercontent.com/Ouranosinc/PAVICS-e2e-workflow-tests/master/downloadrepos
  + chmod a+x downloadrepos
  chmod: cannot access ‘downloadrepos’: No such file or directory
  + wget --quiet https://raw.githubusercontent.com/Ouranosinc/PAVICS-e2e-workflow-tests/master/default_build_params
  + wget --quiet https://raw.githubusercontent.com/Ouranosinc/PAVICS-e2e-workflow-tests/master/binder/reorg-notebooks
  + chmod a+x reorg-notebooks
  chmod: cannot access ‘reorg-notebooks’: No such file or directory
  + wget --quiet --output-document - https://github.com/Ouranosinc/PAVICS-e2e-workflow-tests/archive/master.tar.gz
  + tar xz
  
  gzip: stdin: unexpected end of file
  tar: Child returned status 1
  tar: Error is not recoverable: exiting now
  + ./downloadrepos
  /etc/cron.hourly/PAVICS-deploy-notebooks: line 63: ./downloadrepos: No such file or directory
  + ./reorg-notebooks
  /etc/cron.hourly/PAVICS-deploy-notebooks: line 64: ./reorg-notebooks: No such file or directory
  + mv -v 'PAVICS-e2e-workflow-tests-master/notebooks/*.ipynb' ./
  mv: cannot stat ‘PAVICS-e2e-workflow-tests-master/notebooks/*.ipynb’: No such file or directory
  + rm -rfv PAVICS-e2e-workflow-tests-master
  + rm -rfv downloadrepos default_build_params reorg-notebooks
  + TMP_SCRIPT=/tmp/notebookdeploy.ICk70Vto2LaE/deploy-notebook
  + cat
  + chmod a+x /tmp/notebookdeploy.ICk70Vto2LaE/deploy-notebook
  + docker pull bash
  Using default tag: latest
  latest: Pulling from library/bash
  Digest: sha256:febb3d74f41f2405fe21b7c7b47ca1aee0eda0a3ffb5483ebe3423639d30d631
  Status: Image is up to date for bash:latest
  + docker run --rm --name deploy_tutorial_notebooks -u root -v /tmp/notebookdeploy.ICk70Vto2LaE/deploy-notebook:/deploy-notebook:ro -v /tmp/notebookdeploy.ICk70Vto2LaE/tutorial-notebooks:/tutorial-notebooks:ro -v /data/jupyterhub_user_data:/notebook_dir:rw --entrypoint /deploy-notebook bash
  + cd /notebook_dir
  + rm -rf tutorial-notebooks/WCS_example.ipynb tutorial-notebooks/WFS_example.ipynb tutorial-notebooks/WMS_example.ipynb tutorial-notebooks/WPS_example.ipynb tutorial-notebooks/catalog_search.ipynb tutorial-notebooks/dap_subset.ipynb tutorial-notebooks/esgf-compute-api-examples-devel tutorial-notebooks/esgf-dap.ipynb tutorial-notebooks/finch-usage.ipynb tutorial-notebooks/hummingbird.ipynb tutorial-notebooks/opendap.ipynb tutorial-notebooks/pavics_thredds.ipynb tutorial-notebooks/raven-master tutorial-notebooks/rendering.ipynb tutorial-notebooks/subsetting.ipynb
  + cp -rv '/tutorial-notebooks/*' tutorial-notebooks
  cp: can't stat '/tutorial-notebooks/*': No such file or directory
  + chown -R root:root tutorial-notebooks
  + set +x
  removed directory: ‘/tmp/notebookdeploy.ICk70Vto2LaE/tutorial-notebooks’
  removed ‘/tmp/notebookdeploy.ICk70Vto2LaE/deploy-notebook’
  removed directory: ‘/tmp/notebookdeploy.ICk70Vto2LaE’
  
  notebookdeploy finished START_TIME=2020-04-23T10:01:01-0400
  notebookdeploy finished   END_TIME=2020-04-23T10:02:12-0400
  ```

`1.9.0 <https://github.com/bird-house/birdhouse-deploy/tree/1.9.0>`_ (2020-04-24)
========================================================================================================================

- 
  vagrant: add centos7 and LetsEncrypt SSL cert support, fix scheduler autodeploy remaining issues
  
  Fixes https://github.com/bird-house/birdhouse-deploy/issues/27.
  
  Centos7 support added to Vagrant to reproduce problems found on Medus in PR https://github.com/bird-house/birdhouse-deploy/pull/39 (commit https://github.com/bird-house/birdhouse-deploy/commit/6036dbd5ff072544d902e7b84b5eff361b00f78b):
  
  Problem 1: wget httpS url not working in bash docker image breaking the notebook autodeploy when running under the new scheduler autodeploy: **not reproducible**
  
  Problem 2: all containers are destroyed and recreated when alternating between manually running `./pavics-compose.sh up -d` locally and when the same command is executed automatically by the scheduler autodeploy inside its own container: **not reproducible**
  
  Problem 3: `sysctl: error: 'net.ipv4.tcp_tw_reuse' is an unknown key` on `./pavics-compose.sh up -d` when executed automatically by the scheduler autodeploy inside its own container: **reproduced** but seems **harmless** so **not fixing** it.
  
  Problem 4: current user lose write permission to birdhouse-deploy checkout and other checkout in `AUTODEPLOY_EXTRA_REPOS` when using scheduler autodeploy: **fixed**
  
  Problem 5: no documentation for the new scheduler autodeploy: **fixed**
  
  Another autodeploy fix found while working on this PR: notebook autodeploy broken when `/data/jupyterhub_user_data/tutorial-notebooks` dir do not pre-exist.  Regression from this commit https://github.com/bird-house/birdhouse-deploy/pull/16/commits/6ddaddc74d384299e45b0dc8d50a63e59b3cc0d5 (PR https://github.com/bird-house/birdhouse-deploy/pull/16): before that commit the entire dir was copied, not just the content, so the dir was created automatically.
  
  Centos7 Vagrant box experience is not completely automated as Ubuntu box, even when using the same vagrant-disksize Vagrant plugin as Ubuntu box.  Manual disk resize instruction is provided.  Candidate for automation later if we destroy and recreate Centos7 box very often.  Hopefully the problem is not there for Centos8 so we can forget about this annoyance.
  
  Automatic generation of SSL certificate from LetsEncrypt is also added for both Ubuntu and Centos Vagrant box.  Can be used outside of Vagrant so Medus and Boreas can also benefit next time, if needed.  Later docker image of `certbot` is used so should already be using ACMEv2 protocol (ACMEv1 is being deprecated).
  
  Pagekite is also preserved for both boxes for when exposing port 80 and 443 directly on the internet is not possible but PAVICS still need a real SSL certificate.
  
  Test server: https://lvupavicsmaster.ouranos.ca (Centos7, on internet with LetsEncrypt SSL cert).
  
  Jenkins run only have known errors: http://jenkins.ouranos.ca/job/ouranos-staging/job/lvupavicsmaster.ouranos.ca/4/console
  
  ![2020-04-22-070604_1299x1131_scrot](https://user-images.githubusercontent.com/11966697/79974607-a2707b80-8467-11ea-85b6-3b03f198ce9b.png)

`1.8.10 <https://github.com/bird-house/birdhouse-deploy/tree/1.8.10>`_ (2020-04-09)
========================================================================================================================

- Autodeploy the autodeploy phase 2: everything operational but a few compatibility issues remain
  
  Part of https://github.com/bird-house/birdhouse-deploy/issues/27
  
  Activating the `./components/scheduler` will do everything.  All configurations are centralized in the `env.local` file.
  
  One missing feature is piece-wise choice of platform or notebook autodeploy only, like with the old manual `install-*` stcripts under https://github.com/bird-house/birdhouse-deploy/tree/master/birdhouse/deployment.  Right now it's all or nothing.  I can work on this if you guys think it's needed.
  
  Remaining compatibility issues with Medus (Vagrant box works fine):
  
  * Notebook autodeploy do not work. It looks like using the `bash` docker image, I am unable to wget any httpS address.  This same `docker run` command works fine on my Vagrant box as well.  So there's something on Medus.
  
  ```
  $ docker run --rm --name debug_wget_httpS -u root bash bash -c "wget https://google.com -O -"
  Connecting to google.com (172.217.13.206:443)
  wget: error getting response: Connection reset by peer
  ```
  
  * All the containers are being recreated when `./pavics-compose.sh` runs inside the container (first migration to the new autodeploy mechanism).  To investigate but I suspect this might be due to older version of `docker` and `docker-compose` on Medus.
  
  * This one looks like due to older kernel on Medus:
  ```
  sysctl: error: 'net.ipv4.tcp_tw_reuse' is an unknown key
  sh: 0: unknown operand
  ```
  
  * All the files updated by `git pull` are now owned by `root` (the user inside the container).  I'll have to undo this ownership change, somehow.  This one is super weird, I should have got it on my Vagrant box.  Probably Vagrant did some magic to always ensure files under `/vagrant` is always owned by the user even if changed by user `root`.
  
  * Documentation: update README and list relevant configuration variables in `env.local` for this new `./component/scheduler`.
  
  
  Migrating to this new mechanism requires manual deletion of all the artifacts created by the old install scripts: `sudo rm /etc/cron.d/PAVICS-deploy /etc/cron.hourly/PAVICS-deploy-notebooks /etc/logrotate.d/PAVICS-deploy /usr/local/sbin/triggerdeploy.sh`.  Both can not co-exist at the same time.
  
  Maximum backward-compatibility has been kept with the old existing install scripts style:
  * Still log to the same existing log files under `/var/log/PAVICS`.
  * Old single ssh deploy key is still compatible, but the new mechanism allows for different ssh deploy keys for each extra repos (again, public repos should use https clone path to avoid dealing with ssh deploy keys in the first place)
  * Old install scripts are kept
  
  Features missing in old existing install scripts or how this improves on the old install scripts:
  * Autodeploy of the autodeploy itself !  This is the biggest win.  Previously, if `triggerdeploy.sh` or `PAVICS-deploy-notebooks` script changes, they have to be deployed manually.  It's very annoying.  Now they are volume-mount in so are fresh on each run.
  * `env.local` now drive absolutely everything, source control that file and we've got a true DevOPS pipeline.
  * Configurable platform and notebook autodeploy frequency.  Previously, this means manually editing the generated cron file, less ideal.
  * Do not need any support on the local host other than `docker` and `docker-compose`.  cron/logrotate/git/ssh versions are all locked-down in the docker images used by the autodeploy.  Recall previously we had to deal with git version too old on some hosts.
  * Each cron job run in its own docker image meaning the runtime environment is traceable and reproducible.
  * The newly introduced scheduler component is made extensible so other jobs can added into it as well (ex: backup), via `env.local`, which should source control, meaning all surrounding maintenance related tasks can also be traceable and reproducible.
  
  This is a rather large PR.  For a less technical overview, start with the diff of README.md, env.local.example, common.env.  If a change looks funny to you, read the commit description that introduce that change, the reasoning should be there.

`1.8.9 <https://github.com/bird-house/birdhouse-deploy/tree/1.8.9>`_ (2020-04-08)
========================================================================================================================

- finch: update to 0.5.2
  
  Fix following 2 Jenkins failures:
  
  Tested in this Jenkins run http://jenkins.ouranos.ca/job/ouranos-staging/job/lvupavics-lvu.pagekite.me/20/console
  
  ```
    _________ finch-master/docs/source/notebooks/dap_subset.ipynb::Cell 9 __________
    Notebook cell execution failed
    Cell 9: Cell outputs differ
  
    Input:
    resp = wps.sdii(pr + sub)
    out = resp.get(asobj=True)
    out.output_netcdf.sdii
  
    Traceback:
     mismatch 'text/html'
  
     assert reference_output == test_output failed:
  
      '<pre>&lt;xar...vera...</pre>' == '<pre>&lt;xar...vera...</pre>'
      Skipping 350 identical leading characters in diff, use -v to show
        m/day
      -     cell_methods:   time: mean (interval: 30 minutes)
            history:        pr=max(0,pr) applied to raw data;\n[DATE_TIME] ...
      +     cell_methods:   time: mean (interval: 30 minutes)
            standard_name:  lwe_thickness_of_precipitation_amount
            long_name:      Average precipitation during wet days (sdii)
            description:    Annual simple daily intensity index (sdii) : annual avera...</pre>
  ```
  
  ```
    _________ finch-master/docs/source/notebooks/finch-usage.ipynb::Cell 1 _________
    Notebook cell execution failed
    Cell 1: Cell outputs differ
  
    Input:
    help(wps.frost_days)
  
    Traceback:
     mismatch 'stdout'
  
     assert reference_output == test_output failed:
  
      'Help on meth...ut files.\n\n' == 'Help on meth...ut files.\n\n'
      Skipping 399 identical leading characters in diff, use -v to show
      -    freq : string
      +    freq : {'YS', 'MS', 'QS-DEC', 'AS-JUL'}string
                Resampling frequency
  
            Returns
            -------
            output_netcdf : ComplexData:mimetype:`application/x-netcdf`
                The indicator values computed on the original input grid.
            output_log : ComplexData:mimetype:`text/plain`
                Collected logs during process run.
            ref : ComplexData:mimetype:`application/metalink+xml; version=4.0`
                Metalink file storing all references to output files.
  ```

`1.8.8 <https://github.com/bird-house/birdhouse-deploy/tree/1.8.8>`_ (2020-03-20)
========================================================================================================================

- Jupyter: make configurable public demo user name, passwd, resource limit, login banner
  
  For security reasons, the public demo username and password are not hardcoded anymore.
  
  Compromising of one PAVICS deployment should not compromise all other PAVICS deployments if each deployment use a different password.
  
  The password is set when the public demo user is created in Magpie, see the `birdhouse/README.md` update.
  
  The login banner do not display the public demo password anymore.  If one really want to display the password, can use the top or bottom section of the login banner that is customizable via `env.local`.
  
  Login banner is updated with more notices, please review wording.
  
  Resource limits (only memory limit seems to work with the `DockerSpawner`) is also customizable.
  
  All changes to `env.local` are live after a `./pavics-compose.sh up -d`.
  
  Test server: https://lvupavics-lvu.pagekite.me/jupyter/ (ask me privately for the password :D)

`1.8.7 <https://github.com/bird-house/birdhouse-deploy/tree/1.8.7>`_ (2020-03-19)
========================================================================================================================

- finch: update to v0.5.1

`1.8.6 <https://github.com/bird-house/birdhouse-deploy/tree/1.8.6>`_ (2020-03-16)
========================================================================================================================

- Thredds: New "Datasets" top level for NCML files
  
  http://lvupavics-lvu.pagekite.me/twitcher/ows/proxy/thredds/catalog/datasets/catalog.html (only gridded_obs/nrcan.ncml works on my dev server).
  
  Add a new top-level "Datasets" at the same level as the existing "Birdhouse".
  
  The content of the new top-level comes from `/data/ncml` from the host.  For comparison content of existing "Birdhouse" was coming from `/data/datasets`.

`1.8.5 <https://github.com/bird-house/birdhouse-deploy/tree/1.8.5>`_ (2020-03-13)
========================================================================================================================

- jupyter: update to pavics/workflow-tests:200312 for Raven notebooks

`1.8.4 <https://github.com/bird-house/birdhouse-deploy/tree/1.8.4>`_ (2020-03-10)
========================================================================================================================

- raven: upgrade to pavics/raven:0.10.0

`1.8.3 <https://github.com/bird-house/birdhouse-deploy/tree/1.8.3>`_ (2020-02-17)
========================================================================================================================

- catalog: fix pavicsearch broken due to typo in config
  
  The `thredds_host` should be the exact prefix of each document url found
  in Solr, otherwise it is removed from the search result.
  
  This explains why pavicsearch was returning nothing.
  
  This will fix the `catalog_search.ipynb` notebook that keeps failing on Jenkins.
  
  The typo was introduced in PR
  https://github.com/bird-house/birdhouse-deploy/pull/5, commit
  https://github.com/bird-house/birdhouse-deploy/commit/83c839178fff170dbcb4c4e0586e67d19b9cfbc5

`1.8.2 <https://github.com/bird-house/birdhouse-deploy/tree/1.8.2>`_ (2020-02-10)
========================================================================================================================

- Optionally monitor all components behind Twitcher using canarie api.
  
  Fixes https://github.com/bird-house/birdhouse-deploy/issues/8
  
  The motivation was the need for some quick dashboard for the working state of all the components, not to get more stats.
  
  Right now we bypassing Twitcher, which is not real life, it's not what real users will experience.
  
  This is ultra cheap to add and provide very fast and up-to-date (every minute) result. It's like an always on sanity check that can quickly help debugging any connectivity issues between the components.
  
  It is optional because it assumes all components are publicly accessible.  Might not be the case for everyone.  We can also override the override :D
  
  All components in config/canarie-api/docker_configuration.py.template that do not have public (behind Twitcher) monitoring are added.
  
  Also added Hummingbird and ncWMS2 public monitoring.
  
  @tlogan2000 This will catch accidental Thredds public url breakage like last time and will leverage the existing monitoring on https://pavics.ouranos.ca/canarie/node/service/stats by @moulab88.
  
  @davidcaron @dbyrns This is optional so if the CRIM do not want to enable it, it's fine.
  
  New node monitoring page:
  
  ![Screenshot_2020-02-07 Ouranos - Node Service](https://user-images.githubusercontent.com/11966697/74055606-4a6cc180-49ae-11ea-9cba-887118dbaae6.png)

`1.8.1 <https://github.com/bird-house/birdhouse-deploy/tree/1.8.1>`_ (2020-02-06)
========================================================================================================================

- Increase JupyterHub security.
  
  ab56994 jupyter: limit memory of public user to 500 MB
  90c1950 jupyter: prevent user from loading user-owned config at spawner server startup
  e8f2fa3 jupyter: avoid terminating user running jobs on Hub update
  3f97cc7 jupyter: get ready to prevent browser session re-use even if password changed
  e2ebcc3 jupyter: disable notebook terminal for security reasons

`1.8.0 <https://github.com/bird-house/birdhouse-deploy/tree/1.8.0>`_ (2020-02-03)
========================================================================================================================

- jupyter data migration: touch new location else jupyterhub won't bind mount them
  
  See PR https://github.com/bird-house/birdhouse-deploy/pull/16
  or commit
  https://github.com/bird-house/birdhouse-deploy/commit/53576cc9d36642c50e4a649ca58fc8339559fd4a
  
  See the `if os.path.exists` in the `jupyterhub_config.py`:
  https://github.com/bird-house/birdhouse-deploy/blob/53576cc9d36642c50e4a649ca58fc8339559fd4a/birdhouse/config/jupyterhub/jupyterhub_config.py.template#L36-L48

`1.7.1 <https://github.com/bird-house/birdhouse-deploy/tree/1.7.1>`_ (2020-01-30)
========================================================================================================================

- jupyter: update various packages and add threddsclient
  
  Noticeable changes:
  ```diff
  <     - bokeh==1.4.0
  >   - bokeh=1.4.0=py36_0
  
  <   - python=3.7.3=h33d41f4_1
  >   - python=3.6.7=h357f687_1006
  
  >   - threddsclient=0.4.2=py_0
  
  <     - xarray==0.13.0
  >   - xarray=0.14.1=py_1
  
  <     - dask==2.8.0
  >     - dask==2.9.2
  
  <     - xclim==0.12.2
  >     - xclim==0.13.0
  ```
  
  See PR https://github.com/Ouranosinc/PAVICS-e2e-workflow-tests/pull/34 for more info.

`1.7.0 <https://github.com/bird-house/birdhouse-deploy/tree/1.7.0>`_ (2020-01-22)
========================================================================================================================

- backup solr: should save all of /data/solr, not just the index

