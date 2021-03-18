# Particle Tracking in VPIC

We can select a small fraction of particles in VPIC and track their history in high cadence as the simulation is running. Here I use `mime1836_sigmaic256_bg00/reconnection.cc` as an example. The code is mixed with pieces from different generations. There are many parameters. You can keep most of them as default and modify the following parameters (line 263 - 281 if not specified).

- Change `particle_tracking` to 1 to turn on particle tracking.
- Change `particle_select` to a reasonable number. We will select one particle every `particle_select` particles. If we plan to select about 1 million electrons/ions in the whole simulation, `particle_select` should be about $n_x\times n_y\times n_z\times\texttt{nppc}/10^6$.
- `nframes_per_tracer_file` is the number of time frames saved in one tracer file. We usually set it to 100 to 1000 to reduce the number of IO calls.
- `emf_at_tracer` determines whether electric and magnetic fields are dumped along the particle trajectories. We usually turn it on because the fields are useful for studying particle acceleration.
- `hydro_at_tracer` determines whether hydro fields (e.g., velocity and density) are dumped along the trajectories. Although these quantities can be useful, accumulating hydro fields can be expensive, which may significantly slow down the simulations. We usually turn it off.
- Change `tracer_interval` to some reasonable value (line 496). The choice depends on the physics to study. We usually dump a few thousand time frames of tracer particles. For example, if we want to dump 3000 thousand frames, `tracer_interval` should be around `num_step / 3000`. We can `tracer_interval = 1` to dump every step of the tracer trajectories. The data will be large, but it can be necessary if you are interested in the detailed acceleration processes.
- Each of the tracer particles has a unique ID called `tag` (line 1240 and 1331). As the simulations evolve, particles will move all over the simulation box. `tag` enables particles to be tracked.
- Make sure the lines including (line 2368 and 2390) ` #include "dumptracer_hdf5_single.cc"` are uncommented.

The simulations will write the tracer particle data into `tracer/tracer1` in default. Initially, particle `tag` monotonically increases in the file. But as the simulation evolves, particles are mixed. That's why we need to sort particles afterwords. We can use a parallel sorting code https://github.com/xiaocanli/vpic-sorter to do that. Please see the description there on how to compile the code. To run code, please modify `sort_particles.sh` in the current directory first. You can keep most of them as default and modify the following parameters.
- `trace_particles`: if true, the code will trace `ntraj` particles after particles are sorted.
- `save_sorted_files`: if true, sorted particle data will be saved to files. We can set it to false when, for example, we only want to tracer particle trajectories.
- `run_name`: PIC simulation run name. Better to unique.
- `runpath`: PIC simulation run path.
- `filepath`: the directory where the tracer data is so the code can find the tracer data.
- `particle`: electron or H. It can be other species if the PIC simulation includes them.
- `tstep_min` and `tstep_max`: minimum and maximum time steps.
- `tstep_interval`: time step interval. It is the same as `tracer_interval` in the deck.
- `nsteps`: save as `nframes_per_tracer_file` in the deck.
- `mpi_size`: the number MPI processes for the sorting program.
- `ratio_emax`: maximum energy / starting energy. 1 means to find the highest energy particles. A higher value means to find lower-energy particles.
- `data_dir`: the directory where the trajectory data should be saved. If `trace_particles` is set true, the code will save the trajectory data to `data_dir`. The trajectory file will include `ntraj` tracer trajectories. Please modify the function in `plot_trajectory.py` in the current directory to read and plot the particle trajectories.

This procedure is quite inconvenient. The code was originally designed for sorting tens of million or even billions of tracer particles. For most cases, we only have about 1 million tracer particles. I think we can use python to sort the tracer particles instead, but I don't have a code to that right now. We can read the tracer particle data as numpy arrays sort the data along certain axis.