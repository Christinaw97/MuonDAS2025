# Muon Hands-On Tutorial Session

## Facilitators

- Christina Wang 
- Daniel Wilbern
- Chris Cosby
- Tamer Elkafrawy
- Beren Ozek


## Technical setup

This exercise is designed for, and tested at, the [Purdue Analysis Facility](https://analysis-facility.physics.purdue.edu/en/latest/) (AF). In general, it is expected to run where the following requirements are fulfilled:

* python3
* uproot
* awkward
* ROOT
* NumPy
* matplotlib
* scipy
* Jupyter Notebook
* coffea 
* boost

However, the input files used in the exercises are specific to the Purdue AF and will need to be replaced when running somewhere else. 

Before the live session, please make sure that you can access Purdue Analysis Facility using the instructions below. 

1. Navigate to the [Purdue AF website](https://analysis-facility.physics.purdue.edu/) and click “Login to Purdue Analysis Facility”.
2. On the CILogon page, choose CERN account to log in (using Fermilab or Purdue credentials is also possible).
3. You will be redirected to the “Server Options” page. The default resource selection (4 CPUs, 16 GB RAM) is enough for the HATS exercises, but you can select more resources if needed. **Do not add GPUs to your session – there are not enough GPUs for all participants.**
4. Click “Start” to create your Analysis Facility session. It may take a couple of minutes to load.
5. Done! Your session is ready.

<details>
  <summary>Additional details</summary>
  
- As a CMS member, you can continue using Purdue AF for your work after HATS is over.
- Your Purdue AF session will keep running even if you close your web browser tab, so you can reconnect to it at any time. Idle sessions are terminated after 14 days.
- When you log in for the first time, we create a 25 GB private directory for you at `/home/<username>`, and a shared `/work/users/<username>` directory with 100 GB quota. These directories will persist for 6 months after your last activity at Purdue AF.
- Browse the [documentation](https://analysis-facility.physics.purdue.edu/) to learn more about available functionality.


```
git clone https://github.com/JanFSchulte/MuonHATS.git
cd MuonHATS
```

The repository will appear in the folder structure in the sidebar. It contains four notebooks that can be access simply via this file browser. When starting each of the notebooks, select the `HATS 2024` kernel, which has all required packages installed. When running somewhere else, the included `environment.yml` can be used to create a compatible conda environment. 

## Introduction: General information on muons in CMS

Muons in CMS are maintained and supported by the Muon POG ([Twiki](https://twiki.cern.ch/twiki/bin/viewauth/CMS/MuonPOG)). This group is responsible for the triggering, reconstruction, and identification of muons from the detector signals. The muon detectors, their maintainance and calibration, are the responsibility of the detector projects and Muon DPG. Muons in the L1 trigger are responsibility of the L1 DPG. 

For analysts, the best entry point when looking for information about muons in CMS is currently this [Twiki page](https://twiki.cern.ch/twiki/bin/view/CMS/TWikiPAGsMUO), which contains links to the relevant recommendations. The muon contact of your PAG is your first point of contact for questions about muons, the names are listed on this page as well. Further questions, especially of technical nature, can be asked on [CMS Talk](https://cms-talk.web.cern.ch/c/physics/muo/147). In urgent cases or to request presentations in the one of the POG meetings, contact the POG conveners at `cms-phys-conveners-MUO@cern.ch`. 

Information about muon selections can be found on this [Twiki page](https://twiki.cern.ch/twiki/bin/viewauth/CMS/SWGuideMuonIdRun2) (still to be migrated to a Run 3 version). Efficiencies for standard use cases are provided by the POG, but if custom selections are used, analysts need to derive them themselves, for example using the Tag & Probe tools described [here](https://twiki.cern.ch/twiki/bin/view/CMS/SWGuideMuonTnPOverview). 

## Exercise 1: Introduction, muon object, main variables

In this exercise we will get familiar with the muon objects in a nanoAOD file. We will mostly a NANOAODSIM file, containing simulated Drell–Yan dimuon events generated at NLO with the program MadGraph.

Now please open Exercise-1-Introduction.ipynb. and follow the instructions in the notebook, run and modify the code. In this exercise, we will see how to interact with the NanoAOD content directly, using `uproot` and `awkward` arrays. In the following exercises, we will use the `coffea` framework instead as a higher-level interface to make our life easier. 

After going through the notebook, you should learn the following points.

- how to explore a nanoAOD using uproot
- what muon variables are stored in nanoAOD
- how to do some quick selection using awkward array
- how to perform deltaR matching between generated muons and reconstructed muons, HLT muons and reconstructed muons 

Please try to solve the questions posted in the notebook. Answers can be found in the solutions folder.

## Exercise 2: Muon momentum scale and resolution corrections

The measurement of the transverse momentum of muons is sensitive to several detector conditions:

- the alignment of the tracker and of the muon chambers
- the composition and distribution of the material inside the tracking volume
- the knowledge of the magnetic field inside and outside the solenoid volume. 

All these conditions affect differently the momentum measurement and can produce biases. In particular, the detector misalignment produces a relative bias that generally increases linearly with the momentum. For this reason it is extremely important to have an accurate knowledge of the tracker and muon spectrometer alignment, and a detailed mapping of the detector material and of the magnetic field. Residual biases can be corrected a posteriori, using calibration techniques that generally exploit data from very well-known processes, such as J/ψ→μμ or Z→μμ decays.

In this exercise we will examine the effects produced by these biases in the momentum measurement and we will use correction factors to mitigate them.

Now please open Exercise-2-Muon-momentum-scale-and-resolution-corrections.ipynb. Please follow the instructions on the notebook, run and modify the code. After going through the notebook, you should learn the following points.

- basic steps to plot the Z mass
- how to perform fit using root with various functions
- what is mass resolution and what is transverse momentum resolution, how they are related
- how to perform a momentum scale correction 

Please try to solve the questions posted in the notebook, answers are stored in the solutions folder.

## Exercise 3: Muon identification and isolation

In the previous exercises, we applied cuts on various quantities in muon object without understanding what they were or why we were imposing the suggested requirements. In this exercise, we will build on the discussion in the introduction and the practice with the nanoAOD in the previous exercise to analyze the main properties and quality variables of muon tracks and how they can be used to identify muons from different sources. We will use three seperate NANOAODSIM files, containing simulated Drell–Yan, top-pair, and QCD multijet events:

### Step 1: Isolation variables

To better understand the selection criteria, it is useful to classify each muon according to how it was produced:

- prompt, i.e. from the decay of a W or Z boson or from a τ lepton produced in the hard proton-proton interaction
- heavy flavor decay, i.e. from the decay of a b-quark or c-quark hadron
- light flavor decay, i.e. from the decay of a light-quark hadron, such as a pion or a kaon
- fake muons, such as punch-through or matching of random tracks and muon segments. 

In order to isolate each source of muons, we put cut on various variables. Below are variables commonly used for those cuts; please note that the presented way to access them is used in miniAOD.

- whether the muon is a global muon, i.e. it was reconstructed with a combined fit of tracker and muon chambers measurements
        `muon->isGlobalMuon()` 
- whether the muon is a tracker muon, i.e. it was identified by geometrically matching an inner track with segments in the muon chambers
        `muon->isTrackerMuon()`
- Normalized χ2 of the global track fit
        `if(muon->isGlobalMuon()) muon->globalTrack()->normalizedChi2()`
- Number of muon chamber hits included in the global-muon track fit
        `if(muon->isGlobalMuon()) muon->globalTrack()->hitPattern().numberOfValidMuonHits()`
- Number of muon stations with matched segments
        `muon->numberOfMatchedStations()`
- Number of hits in the pixel detector
        `muon->innerTrack()->hitPattern().numberOfValidPixelHits()`
- Number of hits in the tracker layers
        `muon->innerTrack()->hitPattern().trackerLayersWithMeasurement()`
- Transverse impact parameter of the track with respect to the vertex from which the muon originated
        `muon->muonBestTrack()->dxy(firstGoodVertex->position())`
- Isolation based on the sum of pT of charged-hadron PFCandidates from the leading primary vertex in the event, in a cone of ΔR < 0.4 around the muon
        `muon->pfIsolationR04().sumChargedHadronPt`
- Isolation calculated with neutral-hadron PFCandidates in a cone of ΔR < 0.4 around the muon
        `muon->pfIsolationR04().sumNeutralHadronEt`
- Isolation calculated with photon PFCandidates in a cone of ΔR < 0.4 around the muon
        `muon->pfIsolationR04().sumPhotonEt`
- Isolation calculated with all charged particles in a cone of ΔR < 0.4 around the muon, but not from the leading primary vertex (i.e. pileup contribution to the isolation sum)
        `muon->pfIsolationR04().sumPUPt`
- PF-based combined relative isolation, Δβ-corrected for pileup
        `(muon->pfIsolationR04().sumChargedHadronPt + max(0., mu->pfIsolationR04().sumNeutralHadronEt + mu->pfIsolationR04().sumPhotonEt - 0.5*mu->pfIsolationR04().sumPUPt)) / muon->pt()`
- Tracker-based relative isolation
        `muon->isolationR03().sumPt / muon->pt()`

In the nanoAOD , the following variables can be found:

    Whether the muon is a global muon, i.e. it was reconstructed with a combined fit of tracker and muon chambers measurements
        `Muon_isGlobal` 
    Whether the muon is a tracker muon, i.e. it was identified by geometrically matching an inner track with segments in the muon chambers
        `Muon_isTracker` 
    Number of muon stations with matched segments
        `Muon_nStations` 
    Number of hits in the tracker layers
        `Muon_nTrackerLayers`
    Transverse impact parameter of the track with respect to the vertex from which the muon originated
        `Muon_dxy`
    PF-based combined relative isolation, Δβ-corrected for pileup
        `Muon_pfRelIso04_all` 
    Tracker-based relative isolation
        `Muon_tkRelIso` 

Now, please open Exercise 3 and go to step 1. Please follow the instructions on the notebook, run and modify the code. Please try to solve the questions posted in the notebook, answers are stored in solutions folder.

### Step 2: Standard muon definitions in CMS

The CMS Muon Physics Object Group (MUO POG) takes care of everything that concerns muon reconstruction, identification, high-level triggering, performance evaluation and monitoring, corrections, use in physics analysis, etc. Among other tasks, it develops and maintains a number of standard identification and isolation criteria, which are broadly used in analysis across all CMS. The full list of official definitions can be found on this [Twiki page](https://twiki.cern.ch/twiki/bin/viewauth/CMS/SWGuideMuonIdRun2) page. Here you can find a summary of the most common criteria. These definitions are conveniently implemented as selectors for `reco::Muon` objects in the `MuonSelectors` class, as well as functions in the `pat::Muon` class.

- Loose Muons 

```
bool pat::Muon::isLooseMuon()
bool muon::isLooseMuon(const reco::Muon& muon) {
    return muon.isPFMuon() && (muon.isGlobalMuon() || muon.isTrackerMuon());
}
```
    
- Medium Muons 

```
bool pat::Muon::isMediumMuon()
bool muon::isMediumMuon(const reco::Muon& muon) {
    if( !( isLooseMuon(muon) && muon.innerTrack()-&gt;validFraction()&gt;0.8 )) return false;

    bool goodGlb = muon.isGlobalMuon() &&
        muon.globalTrack()-&gt;normalizedChi2()&lt;3. &&
        muon.combinedQuality().chi2LocalPosition&lt;12. &&
        muon.combinedQuality().trkKink&lt;20.;

    return (segmentCompatibility(muon) &gt; (goodGlb ? 0.303 : 0.451));
}
```

- Tight Muons 

```
bool pat::Muon::isTightMuon(const reco::Vertex& vtx)
bool muon::isTightMuon(const reco::Muon& muon, const reco::Vertex& vtx) {
    if(!muon.isPFMuon() || !muon.isGlobalMuon()) return false;
    bool muID = muon.isGlobalMuon() && muon.globalTrack()-&gt;normalizedChi2()hitPattern().numberOfValidMuonHits()&gt;0 && muon.numberOfMatchedStations()&gt;1;  
    bool hits = muon.innerTrack()-&gt;hitPattern().trackerLayersWithMeasurement()&gt;5 && muon.innerTrack()-&gt;hitPattern().numberOfValidPixelHits()&gt;0;
    bool ip = fabs(muon.muonBestTrack()-&gt;dxy(vtx.position()))dz(vtx.position()))&lt;0.5;

    return muID && hits && ip;
}
```
    
- Soft Muons 

```
bool pat::Muon::isSoftMuon(const reco::Vertex& vtx)
bool muon::isSoftMuon(const reco::Muon& muon, const reco::Vertex& vtx) {
    bool muID = muon::isGoodMuon(muon, TMOneStationTight);
    if(!muID) return false;

    bool layers = muon.innerTrack()-&gt;hitPattern().trackerLayersWithMeasurement()&gt;5 &&
        muon.innerTrack()-&gt;hitPattern().pixelLayersWithMeasurement()&gt;0;
    bool ishighq = muon.innerTrack()-&gt;quality(reco::Track::highPurity);
    bool ip = fabs(muon.innerTrack()-&gt;dxy(vtx.position()))dz(vtx.position()))&lt;20.;

    return layers && ip && ishighq;
}
```
    
- HighPt Muons 

```
bool pat::Muon::isHighPtMuon(const reco::Vertex& vtx)
bool muon::isHighPtMuon(const reco::Muon& muon, const reco::Vertex& vtx){
    bool muID = muon.isGlobalMuon() && muon.globalTrack()-&gt;hitPattern().numberOfValidMuonHits()&gt;0 && (muon.numberOfMatchedStations()&gt;1);
    if(!muID) return false;

    bool hits = muon.innerTrack()-&gt;hitPattern().trackerLayersWithMeasurement()&gt;5 &&
        muon.innerTrack()-&gt;hitPattern().numberOfValidPixelHits()&gt;0;
    bool momQuality = muon.tunePMuonBestTrack()-&gt;ptError()/muon.tunePMuonBestTrack()-&gt;pt()&lt;0.3;
    bool ip = fabs(muon.innerTrack()-&gt;dxy(vtx.position()))dz(vtx.position()))&lt;0.5;

  return muID && hits && momQuality && ip;
}
```

Now, please open Exercise 3 and go to step 2. Please try to solve the questions posted in the notebook, answers are stored in solutions folder.

### Step 3: Muon isolation

Let's now take a detailed look at the isolation variables mentioned at the beginning of Exercise 3. The most common muon isolation algorithm in CMS makes use of the PF candidates found in a region of ΔR < 0.4 around the muon track:

```
mu->pfIsolationR04().sumChargedHadronPt   // pT sum of charged hadrons from the main primary vertex of the event
mu->pfIsolationR04().sumNeutralHadronEt   // pT sum of neutral hadrons
mu->pfIsolationR04().sumPhotonEt          // pT sum of photons
```

In order to exploit the full-detector information, these variables can be combined in a single isolation variable:

```
const reco::MuonPFIsolation &pfR04 = mu->pfIsolationR04();
double combRelIso = (pfR04.sumChargedHadronPt + pfR04.sumNeutralHadronEt + pfR04.sumPhotonE) / mu->pt();   // combined relative isolation
```

The combined isolation turns out to perform better than the individual components separately in terms of efficiency vs background rejection.
Note that for neutral particles (photons and neutral hadrons) it is impossible to determine the vertex they originated from, since they don't have a track. Therefore neutral particles from pileup vertices contribute to the pT sum, and the performance of the combined isolation results to be strongly dependent on the pileup level. Corrections are available to mitigate such effect. The most common in CMS is called "Δβ correction": it estimates the ΣpT of neutral particles coming from pileup vertices using the ΣpT of charged particles from pileup vertices (mu->pfIsolationR04().sumPUPt), and the ratio of neutral-to-charged particles expected in LHC proton-proton collisions. From simulation studies, this ratio results to be about 0.5. 

We can now define a Δβ-corrected combined relative isolation, less sensitive to the number of pileup vertices:

```
const reco::MuonPFIsolation &pfR04 = mu->pfIsolationR04();
double corrCombRelIso = (pfR04.sumChargedHadronPt + std::max(0.0, pfR04.sumNeutralHadronEt + pfR04.sumPhotonEt - 0.5*pfR04.sumPUPt)) / mu->pt();
```

All the variables described above can be find in a minAOD file. In nanoAOD, the Δβ-corrected combined relative isolation is already calculated for you. It is stored as `Muon_pfRelIso04_all`.

Now, please open Exercise 3 and go to step 3. Please try to solve the questions posted in the notebook, answers are stored in solutions folder.

## Exercise 4: Muon efficiency

Tag-and-Probe is a data-driven technique used to calculate lepton reconstruction, identification, and trigger efficiencies. This technique uses narrow dilepton resonances, such as Z (for muons with relatively high pT) or J/ψ (for muons with lower pT). Almost-unbiased estimates of the efficiencies can be obtained at the different stages of muon trigger and offline reconstruction. Events are selected with strict requirements on one muon (the tag), and with a more relaxed selection on the other muon (the probe), such that the probe muon can be used to measure the efficiency in question without large biases. The fraction of probe muons that pass the selection under study gives an estimate of its efficiency. The invariant mass of the tag-probe pair is used to select Z→μμ or J/ψ→μμ events.

The Tag-and-Probe technique is generally used to measure and compare efficiencies in data and in MC simulation, and thus to compute a correction scale factor that can be applied to MC events to match the efficiency observed in data. These scale factors are typically determined as functions of pT and η. If necessary, their dependence on other kinematic variables can be investigated too — e.g. vs the number of vertices, in case of strong pileup dependence. In some cases, customized scale factors are necessary for some analyses, depending on their specific trigger and offline thresholds.

Despite the tight selection on the tag muon and the invariant mass constraints, the selected Z→μμ or J/ψ→μμ sample generally contains background events, which appear as a nonresonant continuum underneath the resonance peak. Therefore the background must be subtracted, to ensure that the efficiency is measured with signal muons only. This can be achieved by fitting the invariant mass spectrum to signal + background shapes (e.g. analytical functions or MC templates). Finding proper functions or templates for signal and background is often the most challenging part of the process.

The total lepton efficiency is generally factorized in multiple steps as follows:
total lepton efficiency = (tracking) × (reconstruction/tracking) × (ID/reconstruction) × (isolation/ID) × (trigger/isolation)
In each efficiency step, the denominator determines the selection of the probe. In the last steps (in part. isolation and trigger), the probe selection is tighter and, therefore, the background level is quite low and the background subtraction is generally easier — or not even needed, in some cases. In this exercise, you will measure efficiencies using simulated Z→μμ events.

Since you are using a pure Z sample, you won't need background subtraction nor fitting. The efficiency will simply be computed by counting probes before and after the selection under study.

You will start by computing "true" efficiencies using the generator-level information. This will be your reference efficiencies. Next, you will implement a simple tag-and-probe algorithm to measure the efficiencies with a data-driven approach, and you will compare your results to the "true" efficiencies. Finally, you will try to use the same algorithm on a real single-muon data sample, taken from the 2022F CMS data.

Detailed instructions are already posted in Exercise-4-Muon-Efficiencies.ipynb. Now please open Exercise-4-Muon-Efficiencies.ipynb. Please follow the instructions on the notebook, run and modify the code. Please try to solve the questions posted in the notebook, answers are stored in solutions folder. 
