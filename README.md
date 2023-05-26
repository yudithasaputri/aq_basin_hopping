# aq_basin_hopping_ANI
How to choose miller_indices in a Cu surface
#!/bin/bash

module purge
module load aseani/12May2020

touch scan.dat
touch all_start.xyz
touch all_final.xyz

rm scan.dat all_start.xyz all_final.xyz



max_x=5
max_y=2

z_shift=0.0




  for beta in 0.0  30.0  60.0  90.0
# for beta in  30.0  60.0  90.0
  do
  
    beta_rad=$(echo "$beta*3.14159/180.0" | bc -l)


      for x_shift in $(seq 0 2 10)
      do
  
         for y_shift in $(seq 0 2 10)
         do  
 
  
          gawk '{  n[NR-1] = $1;
                   x[NR-1] = $2;
                   y[NR-1] = $3;
                   z[NR-1] = $4;
                };
	  
	        END {
	  	  
	        for (i = 0; i < NR; i++)
	        {

                  # z-rotation
                  x2 = x[i] * cos('$beta_rad') - y[i] * sin('$beta_rad');
                  y2 = x[i] * sin('$beta_rad') + y[i] * cos('$beta_rad');
                  z2 = z[i];
     
                  x[i] = x2 + '$x_shift';
                  y[i] = y2 + '$y_shift';
                  z[i] = z2 + '$z_shift';
		  
		  printf "%s   %16.12lf  %16.12lf  %16.12lf\n",n[i], x[i], y[i], z[i];
		}
		
	    }' second_aq_centered.pl  > second_aq_rotated.xyz
	    
	   cat geom.template second_aq_rotated.xyz > geom.xyz

	   natoms=$(head -n1 geom.xyz | gawk '{print $1}')
	   
           tail -n $natoms geom.xyz > geom.pl	   
	   
	   idx=$(./check_dist.gk  geom.pl)
	   
	   if [ $idx == 1 ]; then
	     ase-ani-geoopt.py geom.xyz
	     
	     
	     energy=$(grep energy final_geom.xyz | sed 's/energy=//' | gawk '{ print $15 }') # ani energy

              echo $beta "  " $x_shift  "  " $y_shift "   " $z_shift " " $energy >> scan.dat

 	     cat geom.xyz   >> all_start.xyz
	     cat final_geom.xyz >> all_final.xyz
	   fi

	   
      done
    done 
  done

