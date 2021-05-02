## Documentation for flare-generate-forecast-dev container

### Overview of container functionality

This container is responsible for running the forecast.

### Key software dependencies

### Trigger

This container is triggered after a successful run of `flare-process-observations-dev` container. 

### Inputs and outputs

**Input:** QAQC Files

**Output:** Output files for FLARE forecast in NetCDF Format

### Error logs

### Configuration file

```yaml
### Configuration File for CIBR-FLARE Project
## General Settings
git:
  remote:
    server: github.com
    branch: master
    ssh-key-private:
    user-name:
    user-email:
## Container Settings
container:
  name: flare-process-observations-dev
  working-directory:
    pre-run-pull: FALSE
    post-run-push: FALSE
    git:
      remote:
        server: 192.168.20.20
        port:
        repository: FLARE-forecast/test-data
        branch: master
        directory: fcre

##########################
# Lake information
###########################

lake_name_code: fcre
lake_name: Falling Creek Reservoir
lake_latitude: 37.307   #Degrees North
lake_longitude: 79.837  #Degrees West
local_tzone: "EST"

##########################
#  Information required to generate EML metadata file
##########################

metadata:
    forecast_project_id: test
    generate_eml: TRUE
    abstract: "This is where a longer description of the forest can be added"
    forecast_title: FLARE
    intellectualRights: insert license
    model_description:
        name: General Lake Model
        type: process-based
        repository: https://github.com/AquaticEcoDynamics/GLM/releases/tag/v3.1.0
    me:
        individualName:
              givenName: "Quinn"
              surName: "Thomas"
        electronicMailAddress:  "INSERT"
        id: INSERT ORCID


############################
# Run information
#############################

model_name: glm_aed #other is "null"

base_GLM_nml: configuration_files/glm3.nml
base_AED_nml: .na
base_AED_phyto_pars_nml:  .na
base_AED_zoop_pars_nml:  .na

#All sources of uncertainty and data used to constrain
use_obs_constraint: TRUE
observation_uncertainty: TRUE
process_uncertainty: TRUE
weather_uncertainty: TRUE
initial_condition_uncertainty: TRUE
parameter_uncertainty: TRUE
met_downscale_uncertainty: TRUE
inflow_process_uncertainty: TRUE

#########################
### Depth information
#########################

modeled_depths: [0.10, 0.33, 0.67,
                1.00, 1.33, 1.67,
                2.00, 2.33, 2.67,
                3.00, 3.33, 3.67,
                4.00, 4.33, 4.67,
                5.00, 5.33, 5.67,
                6.00, 6.33, 6.67,
                7.00, 7.33, 7.67,
                8.00, 8.33, 8.67,
                9.00]

##############################
##  EnKF setup
##############################
ensemble_size:  210
localization_distance: .na #distance in meters were covariances in the model error are used
vert_decorr_length: 4.0
no_negative_states: TRUE

#################################
# Parameter calibration information
#################################

par_file: parameter_calibration_config.csv

#####################################
###  Observation information
######################################

obs_config_file: observations_config.csv

#########################################
###  State information
#########################################

states_config_file: states_config.csv

########################################
# Dignostics
#######################################

diagnostics_names:  [extc_coef,
                     rad]

### Initialization and Inflow generation.  Not used by run_enkf_forecast()

use_future_inflow: TRUE
future_inflow_flow_coeff: [0.0010803, 0.9478724, 0.3478991]
future_inflow_flow_error: 0.00965
future_inflow_temp_coeff: [0.20291, 0.94214, 0.04278]
future_inflow_temp_error: 0.943

lake_depth_init: 9.4  #not a modeled state
default_temp_init: [25.667, 24.9101, 23.067, 21.8815, 19.6658, 16.5739, 12.9292, 12.8456, 12.8127, 12.8079, 12.778]
default_temp_init_depths: [0.127, 1.004, 2.005, 3.021, 4.002, 5.004, 6.004, 7.01, 8.001, 9.015, 9.518]
the_sals_init: 0.0
default_snow_thickness_init: 0.0
default_white_ice_thickness_init: 0.0
default_blue_ice_thickness_init: 0.0
```

## [How to Run FLARE Container](https://github.com/FLARE-forecast/FLARE/wiki/How-to-Run-FLARE-Containers)