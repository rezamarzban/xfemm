Building and running a simulation with userland:

```
sudo apt update
sudo apt-get install -y nano octave liboctave-dev g++ git cmake liblapack-dev libblas-dev gnuplot
git clone https://github.com/crobarcro/xfemm.git xfemm
cd xfemm/cfemm/libfemm
cp femmversion.h.in femmversion.h
sed -i 's/@XFEMM_VERSION_MAJOR@/0/g' femmversion.h
sed -i 's/@XFEMM_VERSION_MINOR@/0/g' femmversion.h
sed -i 's/@XFEMM_VERSION_PATCH@/0/g' femmversion.h
sed -i 's/@XFEMM_VERSION_STRING@/"0.0.0dev"/g' femmversion.h
sudo mkdir -p /usr/lib/octave
sudo ln -sf /usr/lib/aarch64-linux-gnu/octave/6.4.0 /usr/lib/octave/6.4.0
```

```
cd ~
nano setup.m
```

Copy and paste this to `setup.m`:
```
clear mex;
cd('xfemm/mfemm');
addpath(genpath(pwd()));
mfemm_setup('ForceMexRecompile', true, 'Verbose', true);
```

Then run:
```
export LIBRARY_PATH="/usr/lib/aarch64-linux-gnu/octave/6.4.0:$LIBRARY_PATH"
octave setup.m
```

```
cd ~
nano sim.m
```

Copy and paste this to `sim.m`:
```
graphics_toolkit("gnuplot");

% Air-cored coil magnetostatics example (FEMM manual tutorial)

FemmProblem = newproblem_mfemm('axi', 'Frequency', 0, 'LengthUnits', 'inches');

outernodes = [0, -4; 0, 4];
[FemmProblem, ~, nodeids] = addnodes_mfemm(FemmProblem, outernodes(:,1), outernodes(:,2));
FemmProblem = addsegments_mfemm(FemmProblem, nodeids(1), nodeids(2));
[FemmProblem, rcsegind] = addarcsegments_mfemm(FemmProblem, nodeids(1), nodeids(2), 180, 'MaxSegDegrees', 2.5);

FemmProblem = addmaterials_mfemm(FemmProblem, '18 AWG');
FemmProblem = addcircuit_mfemm(FemmProblem, 'Coil', 'TotalAmps_re', 1);

CoilBlockProps.BlockType = '18 AWG';
CoilBlockProps.InCircuit = 'Coil';
CoilBlockProps.Turns = 1000;
CoilBlockProps.MaxArea = 0.1;
FemmProblem = addrectregion_mfemm(FemmProblem, 0.5, -1, 1, 2, CoilBlockProps);

FemmProblem = addblocklabel_mfemm(FemmProblem, 0.5, 1.5, 'BlockType', 'Air', 'MaxArea', 0.1);

mu_0 = 4*pi*1e-7; R = 4;
[FemmProblem, ~, boundname] = addboundaryprop_mfemm(FemmProblem, 'ABC', 2, 'c0', 1/(mu_0*R), 'c1', 0);
FemmProblem.ArcSegments(rcsegind).BoundaryMarker = boundname;

% Geometry plot
figure(1); plotfemmproblem(FemmProblem); title('Problem Geometry');
print('/home/userland/geometry.png', '-dpng', '-r150');

% Solve
filename = '/home/userland/mag_tutorial.fem';
writefemmfile(filename, FemmProblem);
filename = fmesher(filename);
ansfile = fsolver(filename);

% Post-process
myfpproc = fpproc();
myfpproc.opendocument(ansfile);

disp('Field at center (0,0):'); center_vals = myfpproc.getpointvalues(0,0); disp(center_vals);

% Axial line (r=0)
z = linspace(-4,4,200)'; r = zeros(size(z));
p = myfpproc.getpointvalues(r, z);
Bz = p(4,:)'; Bmag = p(2,:)';
figure(2); plot(z, Bz, 'b', 'linewidth', 2); hold on; plot(z, Bmag, 'k--');
xlabel('z (inches)'); ylabel('B (T)'); title('B_z and |B| along axis (r=0)');
legend('B_z', '|B|'); grid on;
print('/home/userland/axial_field.png', '-dpng', '-r150');

% Midplane (z=0)
r = linspace(0,4,200)'; z = zeros(size(r));
p = myfpproc.getpointvalues(r, z);
Br = p(3,:)'; Bz = p(4,:)'; Bmag = p(2,:)';
figure(3); plot(r, Bz, 'b', 'linewidth', 2); hold on;
plot(r, Br, 'r', 'linewidth', 2); plot(r, Bmag, 'k--', 'linewidth', 2);
xlabel('r (inches)'); ylabel('B (T)'); title('Field in midplane (z=0)');
legend('B_z', 'B_r', '|B|'); grid on;
print('/home/userland/midplane_field.png', '-dpng', '-r150');
```

Then run:
```
octave --eval "addpath(genpath('~/xfemm/mfemm')); source('~/sim.m');"
```

* Note: If Octave version changes, adjust `6.4.0` accordingly (check with `octave-config -v`).
