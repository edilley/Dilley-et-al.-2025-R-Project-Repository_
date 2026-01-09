# Dockerfile
FROM rocker/r-ver:4.4.0

# ---- System dependencies (common for R + compiled packages + graphics stack) ----
RUN apt-get update && apt-get install -y \
    ca-certificates \
    curl \
    wget \
    git \
    pandoc \
    build-essential \
    gfortran \
    cmake \
    pkg-config \
    zlib1g-dev \
    libpng-dev \
    libjpeg-dev \
    libtiff5-dev \
    libfontconfig1-dev \
    libfreetype6-dev \
    libharfbuzz-dev \
    libfribidi-dev \
    libcurl4-openssl-dev \
    libssl-dev \
    libxml2-dev \
    libicu-dev \
    libnlopt-dev \
    libgsl-dev \
    && rm -rf /var/lib/apt/lists/*

# ---- renv configuration for Docker (CRITICAL) ----
# - disable sandbox so packages install into the project library
# - disable autoloader so .Rprofile can't bootstrap at runtime
# - force the project library path explicitly
ENV RENV_CONFIG_PAK_ENABLED=FALSE \
    RENV_CONFIG_SANDBOX_ENABLED=FALSE \
    RENV_CONFIG_AUTOLOADER_ENABLED=FALSE \
    RENV_PATHS_LIBRARY=/project/renv/library \
    TZ=UTC \
    LANG=C.UTF-8 \
    LC_ALL=C.UTF-8

WORKDIR /project

# ---- Copy project first (except ignored files via .dockerignore) ----
# IMPORTANT: you must have a .dockerignore that excludes renv/library and renv/staging
COPY . .

# ---- Ensure project renv library exists ----
RUN mkdir -p /project/renv/library

# ---- Install renv + restore packages at BUILD TIME (never at runtime) ----
RUN R --vanilla -e "install.packages('renv', repos='https://cloud.r-project.org')"
RUN R --vanilla -e "renv::restore(project='/project', prompt=FALSE, clean=TRUE); renv::status('/project')"

# ---- Runtime: load the already-restored library and render ----
CMD ["R", "--vanilla", "-e", "renv::load('/project'); rmarkdown::render('Dilley et al. 2025_PeerJ_Final.Rmd', output_dir='outputs')"]
