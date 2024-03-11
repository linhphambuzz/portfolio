---
layout: post
title:  "Neutrino-Argon target reconstruction simulations"
date:   2024-03-09 12:31:00 -0600
category: project
show: Internship Project - Event Reconstructions
description: Simulation of Argon target in Neutrio-Argon interactions coupled with using Monte Carlos-based simulations to evaluate reconstruction processes between theoretical models for cross-section measurements.
skills: [C++, Object Oriented Programming]
no: 2
---


My oral presentation of this project secured the first place at the [Gulf Coast Undergraduate Symposium](https://gcurs.rice.edu/), hosted by Rice University in 2020. 

- [Official poster](https://lss.fnal.gov/archive/2020/poster/fermilab-poster-20-097-scd.pdf)
- [Report paper](https://drive.google.com/file/d/1TmzhN8C2uvW_wpQIbdKUbC8oqhjt2y34/view?usp=sharing)

## Objectives 

- Average detector simulations were computed using ROOT- a data analysis framework developed by CERN, from the kinematic variable imbalances of outgoing products in Neutrino-Argon interaction event reconstructions. 
- [GENIE](https://github.com/GENIE-MC)- a Monte Carlo-based simulation for event generators was employed to evaluate event reconstruction processes between two distinct models of the Uboonecode analysis framework. 
- Differential cross section measurement were computed from final-state particles kinematic variables. 

## What did I do? 

Below are the highlight of what I did for the project, presented in no particular order. 

### **Computed Mitigation Matrix**



### **Extracted Kinematic Varibles** 


- Fetch the models from GENIE 

{% highlight C++ %}
TFile* default_model = new TFile("MicroBooNE_G18_10a_02_11a_numu_CCinclMEC.gst.root","read");
TFile* alternate_model = new TFile("MicroBooNE_G00_00b_00_000_numu_CCinclMEC.gst.root","read"); 

//set ptr for default and alternate model objects
default_model->GetObject("gst",def_ptr);
alternate_model->GetObject("gst",al_ptr);

//get numbers of events/collisions
long entries = def_ptr -> GetEntries();  
{% endhighlight %}

- Extract momentum from all muon particles. 

{% highlight C++ %}
TVector3 p3p ,p3mu; 
double pxl = 0, pyl = 0, pzl = 0; //initialize muon momentum dimensions 
def_ptr->SetBranchAddress("pxl", &pxl);
def_ptr->SetBranchAddress("pyl", &pyl);
def_ptr->SetBranchAddress("pzl", &pzl);

for (int i= 0; i< numb_ent; i++){
    def_ptr-> GetEntry(i);
    //Analyze dimensions of muon momentum
    p3mu.SetX(pxl);  
    p3mu.SetY(pyl); 
    p3mu.SetZ(pzl);

    //Calculating muon polar angle
    mpa = pzl/ p3mu.Mag();
}


{% endhighlight %}


- Extract only proton with the largest momentum 

{% highlight C++ %}
int nf =0; //number of final state particles in the hadronic system
def_ptr->SetBranchAddress("nf",&nf);

for(int j=0; j< nf ; j++)
{

    if (pdgf[j] == 2212)
    {
            isProton= true;
            p3p.SetX(pxf[j]);
            p3p.SetY(pyf[j]);
            p3p.SetZ(pzf[j]);

            if (p3p.Mag()>mag_p) //only take the bigger value
            {
                mag_p = p3p.Mag();
                a = j;  //a is the track index of the event which has the highest momentum
            };
    };

}
{% endhighlight %}


- Compute Single Transverse Variables 

Single Transverse Variables are defined on a transverse plane which is oriented perpendicular to the direction of the incoming neutrino. More details could be found in my linked report paper above.

{% highlight C++ %}
void compute_stvs( const TVector3& p3mu, const TVector3& p3p, float& delta_pT, float& delta_phiT, float& delta_alphaT, float& delta_pL, float& pn )

{
  delta_pT = (p3mu + p3p).Perp();

  delta_phiT = std::acos( (-p3mu.X()*p3p.X() - p3mu.Y()*p3p.Y()) / (p3mu.XYvector().Mod() * p3p.XYvector().Mod()) );

  TVector2 delta_pT_vec = (p3mu + p3p).XYvector();

  delta_alphaT = std::acos( (-p3mu.X()*delta_pT_vec.X()- p3mu.Y()*delta_pT_vec.Y()) / (p3mu.XYvector().Mod() * delta_pT_vec.Mod()) );
}


{% endhighlight %}

This function is used to compute all the necessary variables for cross-sections measurement, from muon and the leading proton momentum. 

### **Computed cross-section measurements**


- Computed the cross-section 

{% highlight C++ %}
int A_TARGET = 40;
const double FLUX_INTEGRAL_RELATIVE_TOLERANCE = 1e-8;

double total_xsec( const genie::GEVGDriver& evg_driver, double Ev )
{
  // Get the list of available interactions from the event generation driver
  const genie::InteractionList* ilist = evg_driver.Interactions();

  double xsec_sum = 0.;

  for ( const auto* interaction : *ilist ) {
    const genie::Spline* spl = evg_driver.XSecSpline( interaction );
    xsec_sum += spl->Evaluate( Ev ) / genie::units::cm2;
  }

  return xsec_sum;
}
{% endhighlight %}

- Normalize the histograms for plotting

{% highlight C++ %}


// Normalize the flux histograms to unity to use for cross section weighting
// numu is short for neutrino muon 
double flux_integral_numu = flux_hist_numu->Integral( "width" );
flux_hist_numu->Scale( 1. / flux_integral_numu );
double max_energy_numu = flux_hist_numu->GetBinLowEdge( flux_hist_numu->GetNbinsX() + 1 );


// Functions to use for numerical integration. Element x[0] is the neutrino energy
TF1 xsec_weighted_flux_func_numu("temp_func", [&](double* x, double*)
{
    int flux_bin = flux_hist_numu->FindBin( x[0] );
    double flux = flux_hist_numu->GetBinContent( flux_bin );

    double xsec = total_xsec( evgd_numu, x[0] );

    return flux * xsec;
}, 0., max_energy_numu, 0);

double flux_avg_xsec_numu = xsec_weighted_flux_func_numu.Integral(0., max_energy_numu, FLUX_INTEGRAL_RELATIVE_TOLERANCE);

{% endhighlight %}


## Summary 
- The majority of the time I spent on this project was to learn about the objects offered in ROOT, so that I can appropriately extract the parameters that I needed. 
- While the project is primarily rooted in theoretical physics, I gained valuable experience in programming and comprehending C++.