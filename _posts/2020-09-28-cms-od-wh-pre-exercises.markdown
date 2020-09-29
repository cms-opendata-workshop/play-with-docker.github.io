---
layout: post
title:  "CMS Open data workshop pre-exercise cheat sheet"
author: "@katilp"
date:   2020-09-28
categories: beginner
tags: [docker,cmssw]
---

The CMS open data workshop is starting and you are not yet done with the pre-exercises? The dog ate the pre-exercise assignment? Your docker container from the first pre-exercise set is still downloading? You run out of disk space on your laptop? That's bad as you really need a working environment to work on CMS open data.

No worries, however, we have a tool for you for this workshop, thanks LPC! On this page, you can run most of the pre-exercises in a ready-made docker installation. You should still read through the tutorial carefully. This is just a cheat sheet to catch the train if you're late, the explanations are not repeated.

Note also that this setup will only be available during the workshop, and whatever you type in or run will be lost after the session. Make sure to contact the facilitators to get your own setup up and running so that you can continue working with CMS data also after the wokshop.

---

Concepts in this exercise:
* Open the CMS open data container
* Build and run a simple CMSSW analysis job
* Access a physics object collection in an analysis job
* Configure a CMSSW analysis job

---

Tips:

Code snippets are shown in one of three ways throughout this environment:

1. Code that looks like `this` is sample code snippets that is usually part of an explanation.
2. Code that appears in box like the one below can be clicked on and it will automatically be typed in to the appropriate terminal window:
```.term1
uname -a
```
3. Code appearing in windows like the one below is code that you should type in yourself. Usually there will be a unique ID or other bit your need to enter which we cannot supply. Items appearing in <> are the pieces you should substitute based on the instructions.
```
docker container start <container ID>
```

## 1.0 Running the CMS open data container
The [first pre-exercise set](https://cms-opendata-workshop.github.io/workshop-lesson-docker-preexercises/index.html) gives you an introduction to docker and how to use the CMS open data container images. If you are in this page, it is likely that you have encountered problems with setting up your docker environment or running the CMS open data container, and you do not have the [VM setup](https://cms-opendata-workshop.github.io/workshop-lesson-virtualmachine/) up and running either. 

Here, you have a ready-made environment, which you can run in the screen to the right. Start by opening the CMS open data container (see all details in the tutorial section [Using docker with the CMS open data](https://cms-opendata-workshop.github.io/workshop-lesson-docker-preexercises/03-testing-docker-for-cms-opendata/index.html)):
```.term1
docker run -it --name myopendataproject --volume "/cvmfs:/cvmfs" cmsopendata/cmssw_5_3_32 /bin/bash
```

It will start downloading and when ready you will see the following output:
```
root@node1:~# docker run -it --name myopendataproject --volume "/cvmfs:/cvmfs" cmsopendata/cmssw_5_3_32 /bin/bash
Unable to find image 'cmsopendata/cmssw_5_3_32:latest' locally
latest: Pulling from cmsopendata/cmssw_5_3_32
e8114d4b0d10: Pull complete
a3eda0944a81: Pull complete
a88502447863: Pull complete
Digest: sha256:6b9a12992ba088a168b87df98a841d3c56dede326684f5551368fd359acfb43c
Status: Downloaded newer image for cmsopendata/cmssw_5_3_32:latest
Setting up CMSSW_5_3_32
CMSSW should now be available.
```

This is now your CMS open data working environment in a docker container. You can exit from the container with
```.term1
exit
```
and return to it with
```.term1
docker start -i myopendataproject
```

## 1.1 Test and validate with a simple CMS analysis job
Do as explained in tutorial section [Test and validate](https://cms-opendata-workshop.github.io/workshop-lesson-docker-preexercises/05-validation/index.html). Read it through carefully, this is only a listing of the commands to run in your docker terminal. This exercise requires some editing. You can do it on the screen to the right using the basic `vi` editor, or work in a separate page  http://docker.cms-cloud.ml/  (if you change to that page, remember to start the container first with `docker run -it --name myopendataproject --volume "/cvmfs:/cvmfs" cmsopendata/cmssw_5_3_32 /bin/bash`)

```.term1
mkdir Demo
cd Demo
mkedanlzr DemoAnalyzer
cd ../
scram b
```

Follow the instructions in [the tutorial](https://cms-opendata-workshop.github.io/workshop-lesson-docker-preexercises/05-validation/index.html) to edit `Demo/DemoAnalyzer/demoanalyzer_cfg.py`. To use the `vi` editor:

```.term1
vi Demo/DemoAnalyzer/demoanalyzer_cfg.py
```

enter the edit mode typing `i`, do your edits, and save with `esc` `:wq`. 

Replace `file:myfile.root` with `root://eospublic.cern.ch//eos/opendata/cms/Run2011A/ElectronHad/AOD/12Oct2013-v1/20001/001F9231-F141-E311-8F76-003048F00942.root` to point to an example file.

Change also the maximum number of events to 10.  I.e., change `-1`to `10` in `process.maxEvents = cms.untracked.PSet( input = cms.untracked.int32(-1))`.

If you do not get the editing to work, or if you mess up everything, you can modify the `demoanalyzer_cfg.py` file by clicking in the following in the terminal to the right. This work-around is nothing to be proud of, but it works :woman_shrugging: 

```.term1
cat > Demo/DemoAnalyzer/demoanalyzer_cfg.py <<EOL     
import FWCore.ParameterSet.Config as cms

process = cms.Process("Demo")

process.load("FWCore.MessageService.MessageLogger_cfi")

process.maxEvents = cms.untracked.PSet( input = cms.untracked.int32(10) )

process.source = cms.Source("PoolSource",
    # replace 'myfile.root' with the source file you want to use
    fileNames = cms.untracked.vstring(
        'root://eospublic.cern.ch//eos/opendata/cms/Run2011A/ElectronHad/AOD/12Oct2013-v1/20001/001F9231-F141-E311-8F76-003048F00942.root'
    )
)

process.demo = cms.EDAnalyzer('DemoAnalyzer'
)


process.p = cms.Path(process.demo)
EOL
```

Run the job with:

```.term1
cmsRun Demo/DemoAnalyzer/demoanalyzer_cfg.py
```

## 1.2 Access a physics object collection in an analysis job
The [tutorial set on CMSSW](https://cms-opendata-workshop.github.io/workshop-lesson-cmssw/index.html) gives a good introduction to some functionalities of the CMSSW which will be useful in this workshop. Read it through carefully. Again, as for the docker tutorial set, here we **only** list the commands, but do not repeat the instructions. Clicking in the commands is good for catching up, but to learn, go back to the tutorial pages. 

To get access to a physics object collection, follow [this episode](https://cms-opendata-workshop.github.io/workshop-lesson-cmssw/04_source/index.html) of the tutorial.

To add a muon collection, you need to modify the source code in `Demo/DemoAnalyzer/src/DemoAnalyzer.cc` and `Demo/DemoAnalyzer/BuildFile.xml`.

Again, in case of trouble, modify the files with 

```.term1
cat > Demo/DemoAnalyzer/src/DemoAnalyzer.cc <<EOL     
// -*- C++ -*-
//
// Package:    DemoAnalyzer
// Class:      DemoAnalyzer
// 
/**\class DemoAnalyzer DemoAnalyzer.cc Demo/DemoAnalyzer/src/DemoAnalyzer.cc

 Description: [one line class summary]

 Implementation:
     [Notes on implementation]
*/
//
// Original Author:  
//         Created:  Tue Sep 29 09:11:01 CEST 2020
// $Id$
//
//
// system include files
#include <memory>

// user include files
#include "FWCore/Framework/interface/Frameworkfwd.h"
#include "FWCore/Framework/interface/EDAnalyzer.h"

#include "FWCore/Framework/interface/Event.h"
#include "FWCore/Framework/interface/MakerMacros.h"

#include "FWCore/ParameterSet/interface/ParameterSet.h"


//classes to extract Muon information
#include "DataFormats/MuonReco/interface/Muon.h"
#include "DataFormats/MuonReco/interface/MuonFwd.h"

#include<vector>

//
// class declaration
//

class DemoAnalyzer : public edm::EDAnalyzer {
   public:
      explicit DemoAnalyzer(const edm::ParameterSet&);
      ~DemoAnalyzer();

      static void fillDescriptions(edm::ConfigurationDescriptions& descriptions);


  private:
     virtual void beginJob() ;
     virtual void analyze(const edm::Event&, const edm::EventSetup&);
     virtual void endJob() ;

     virtual void beginRun(edm::Run const&, edm::EventSetup const&);
     virtual void endRun(edm::Run const&, edm::EventSetup const&);
     virtual void beginLuminosityBlock(edm::LuminosityBlock const&, edm::EventSetup const&);
     virtual void endLuminosityBlock(edm::LuminosityBlock const&, edm::EventSetup const&);

     // ----------member data ---------------------------
    
     std::vector<float> muon_e; //energy values for muons in the event
};

//
// constants, enums and typedefs
//
//

//
// static data member definitions
//

//
// constructors and destructor
//
DemoAnalyzer::DemoAnalyzer(const edm::ParameterSet& iConfig)

{
   //now do what ever initialization is needed

}


DemoAnalyzer::~DemoAnalyzer()
{
 
   // do anything here that needs to be done at desctruction time
   // (e.g. close files, deallocate resources etc.)

}


//
// member functions
//

// ------------ method called for each event  ------------
void DemoAnalyzer::analyze(const edm::Event& iEvent, const edm::EventSetup& iSetup)
{
 using namespace edm;
 
 //clean the container
 muon_e.clear();
 
 //define the handler and get by label
 Handle<reco::MuonCollection> mymuons;
 iEvent.getByLabel("muons", mymuons);

 //if collection is valid, loop over muons in event
 if(mymuons.isValid()){
    for (reco::MuonCollection::const_iterator itmuon=mymuons->begin(); itmuon!=mymuons->end(); ++itmuon){
        muon_e.push_back(itmuon->energy());
    }
 }

 //print the vector
 for(unsigned int i=0; i < muon_e.size(); i++){
    std::cout <<"Muon # "<<i<<" with E = "<<muon_e.at(i)<<" GeV."<<std::endl;
 }

#ifdef THIS_IS_AN_EVENT_EXAMPLE
  Handle<ExampleData> pIn;
  iEvent.getByLabel("example",pIn);
#endif

#ifdef THIS_IS_AN_EVENTSETUP_EXAMPLE
  ESHandle<SetupData> pSetup;
  iSetup.get<SetupRecord>().get(pSetup);
#endif
}

// ------------ method called once each job just before starting event loop  ------------
void 
DemoAnalyzer::beginJob()
{
}

// ------------ method called once each job just after ending the event loop  ------------
void 
DemoAnalyzer::endJob() 
{
}

// ------------ method called when starting to processes a run  ------------
void 
DemoAnalyzer::beginRun(edm::Run const&, edm::EventSetup const&)
{
}

// ------------ method called when ending the processing of a run  ------------
void 
DemoAnalyzer::endRun(edm::Run const&, edm::EventSetup const&)
{
}

// ------------ method called when starting to processes a luminosity block  ------------
void 
DemoAnalyzer::beginLuminosityBlock(edm::LuminosityBlock const&, edm::EventSetup const&)
{
}

// ------------ method called when ending the processing of a luminosity block  ------------
void 
DemoAnalyzer::endLuminosityBlock(edm::LuminosityBlock const&, edm::EventSetup const&)
{
}

// ------------ method fills 'descriptions' with the allowed parameters for the module  ------------
void
DemoAnalyzer::fillDescriptions(edm::ConfigurationDescriptions& descriptions) {
  //The following says we do not know what parameters are allowed so do no validation
  // Please change this to state exactly what you do use, even if it is no parameters
  edm::ParameterSetDescription desc;
  desc.setUnknown();
  descriptions.addDefault(desc);
}

//define this as a plug-in
DEFINE_FWK_MODULE(DemoAnalyzer);
EOL
```

```.term1
cat > Demo/DemoAnalyzer/BuildFile.xml <<EOL     
<use name="FWCore/Framework"/>
<use name="FWCore/PluginManager"/>
<use name="DataFormats/MuonReco"/>
<use name="FWCore/ParameterSet"/>
<flags EDM_PLUGIN="1"/>
<export>
   <lib name="1"/>
</export>
EOL
```

Compile with
```.term1
scram b
```

and run:
```.term1
cmsRun Demo/DemoAnalyzer/demoanalyzer_cfg.py
```

## 1.3 Configuring a CMSSW analysis job
The [CMSSW tutorial section on the configuration](https://cms-opendata-workshop.github.io/workshop-lesson-cmssw/05_configuration/index.html) explains how to change some parameter in the configuration file.

You can change the output frequency with the `MessageLogger` parameters. Add the line `process.MessageLogger.cerr.FwkReport.reportEvery = 5` after the `load` line in `Demo/DemoAnalyzer/demoanalyzer_cfg.py`. Increase the number of events to 100. If you have problmes with editors, use the following

```.term1
cat > Demo/DemoAnalyzer/demoanalyzer_cfg.py <<EOL     
import FWCore.ParameterSet.Config as cms

process = cms.Process("Demo")

process.load("FWCore.MessageService.MessageLogger_cfi")
process.MessageLogger.cerr.FwkReport.reportEvery = 5

process.maxEvents = cms.untracked.PSet( input = cms.untracked.int32(100) )

process.source = cms.Source("PoolSource",
    # replace 'myfile.root' with the source file you want to use
    fileNames = cms.untracked.vstring(
        'root://eospublic.cern.ch//eos/opendata/cms/Run2011A/ElectronHad/AOD/12Oct2013-v1/20001/001F9231-F141-E311-8F76-003048F00942.root'
    )
)

process.demo = cms.EDAnalyzer('DemoAnalyzer'
)


process.p = cms.Path(process.demo)
EOL
```

Run the job with:
```.term1
cmsRun Demo/DemoAnalyzer/demoanalyzer_cfg.py
```
