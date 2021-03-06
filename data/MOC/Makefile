# This Makefile executes the steps detailed in Chapter 3 of the
# Stereo Pipeline Users' Guide

# Users with custom paths must override this variable
# EXECDIR  == executable directory
# EEXECDIR == executable directory if in a result dir. (For handling
#             relative paths)
EXECDIR ?= ../../bin/
ifeq ($(EXECDIR),)
	EEXECDIR=$(EXECDIR)
else
	EEXECDIR=../$(EXECDIR)
endif

# Executables
STEREO = $(EXECDIR)stereo
IMAGE2QTREE = $(EXECDIR)image2qtree
POINT2DEM = $(EEXECDIR)point2dem
ORTHOPROJECT = $(EEXECDIR)orthoproject
HILLSHADE = $(EEXECDIR)hillshade
COLORMAP = $(EEXECDIR)colormap
DISPARITYDEBUG = $(EEXECDIR)disparitydebug
POINT2MESH = $(EEXECDIR)point2mesh

# Output variables
RESULTSDIR = map
PREFIX = $(RESULTSDIR)/$(RESULTSDIR)
POINTCLOUD = $(PREFIX)-PC.tif

RESULTSDIRU = nonmap
PREFIXU = $(RESULTSDIRU)/$(RESULTSDIRU)
POINTCLOUDU = $(PREFIXU)-PC.tif

all: products_map products_nonmap

# Pattern rule to create .cub files from .imq files
%.cub: %.imq
	mocproc from= $< to= $@ Mapping= NO

jigsaw: M0100115.cub E0201461.cub
	echo M0100115.cub > cube.lis; \
	echo E0201461.cub >> cube.lis; \
	jigsaw fromlist=cube.lis radius=yes update=yes twist=no cnet=control.net onet=control_jigsaw.net spsolve=position

M0100115.map.cub: jigsaw
	cam2map from=M0100115.cub to=M0100115.map.cub

E0201461.map.cub: jigsaw M0100115.map.cub
	cam2map from=E0201461.cub to=E0201461.map.cub map=M0100115.map.cub matchmap=true

M0100115.map/M0100115.kml: M0100115.map.cub
	$(IMAGE2QTREE) -m kml --normalize $<

E0201461.map/E0201461.kml: E0201461.map.cub
	$(IMAGE2QTREE) -m kml --normalize $<

$(POINTCLOUD): M0100115.map.cub E0201461.map.cub
	mkdir -p $(RESULTSDIR); \
	$(STEREO) $^ $(PREFIX) -s stereo.map

$(PREFIX)-F-H.tif: $(POINTCLOUD)
	cd $(RESULTSDIR); $(DISPARITYDEBUG) $(RESULTSDIR)-F.tif

$(PREFIX).ive: $(POINTCLOUD)
	cd $(RESULTSDIR); $(POINT2MESH) $(RESULTSDIR)-PC.tif $(RESULTSDIR)-L.tif

$(PREFIX)-DEM.tif: $(POINTCLOUD)
	cd $(RESULTSDIR); $(POINT2DEM) -r mars --orthoimage $(RESULTSDIR)-L.tif -n --error --nodata -32767 $(RESULTSDIR)-PC.tif;

$(PREFIX)-OrthoR.tif: $(PREFIX)-DEM.tif
	cd $(RESULTSDIR); $(ORTHOPROJECT) $(RESULTSDIR)-DEM.tif ../M0100115.cub $(RESULTSDIR)-OrthoR.tif;

$(PREFIX)-OrthoL.tif: $(PREFIX)-DEM.tif
	cd $(RESULTSDIR); $(ORTHOPROJECT) $(RESULTSDIR)-DEM.tif ../E0201461.cub $(RESULTSDIR)-OrthoL.tif;

$(PREFIX)-DEM_CMAP.tif: $(PREFIX)-DEM.tif
	cd $(RESULTSDIR); \
	$(HILLSHADE) $(RESULTSDIR)-DEM.tif; \
	$(COLORMAP) -s $(RESULTSDIR)-DEM_HILLSHADE.tif $(RESULTSDIR)-DEM.tif; \
	$(COLORMAP) $(RESULTSDIR)-DEMError.tif --lut ../../TriangulationError.lut

products_map: $(PREFIX)-F-H.tif $(PREFIX).ive $(PREFIX)-OrthoL.tif $(PREFIX)-OrthoR.tif E0201461.map/E0201461.kml M0100115.map/M0100115.kml $(PREFIX)-DEM_CMAP.tif

$(POINTCLOUDU): jigsaw
	mkdir -p $(RESULTSDIRU)
	$(STEREO) M0100115.cub E0201461.cub $(PREFIXU) -s stereo.nonmap

$(PREFIXU).ive: $(POINTCLOUDU)
	cd $(RESULTSDIRU); $(POINT2MESH) $(RESULTSDIRU)-PC.tif $(RESULTSDIRU)-L.tif

nonmap_point2dem: $(POINTCLOUDU)
	cd $(RESULTSDIRU); \
	$(POINT2DEM) -r mars --orthoimage $(RESULTSDIRU)-L.tif $(RESULTSDIRU)-PC.tif --error --nodata -32767 -n; \
	$(HILLSHADE) $(RESULTSDIRU)-DEM.tif; \
	$(COLORMAP) -s $(RESULTSDIRU)-DEM_HILLSHADE.tif $(RESULTSDIRU)-DEM.tif; \
	$(COLORMAP) $(RESULTSDIRU)-DEMError.tif --lut ../../TriangulationError.lut

products_nonmap: $(PREFIXU).ive nonmap_point2dem

clean:
	rm -rf $(RESULTSDIR) $(RESULTSDIRU)
	rm -rf M0100115.map E0201461.map cube.lis control_ba.net *.match bundleout* residuals.csv control_jigsaw.net

distclean: clean
	rm -rf print.prt *.cub *.vwip *.match *~
