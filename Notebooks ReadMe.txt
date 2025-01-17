Fura_ROI_Detection_Automated

Purpose:
The purpose of this notebook is to automatically detect and draw ROIs around individual cells in 340nm
Tif movie files. These ROIs are then used to gather fluorescence data per pixel over time from 340/380nm Tif
movie files. The plotted 340/380nm fluorescence over time is then outputted for each ROI along with an excel
file containing the total data for all ROIs. This notebook is meant to be used prior to use of the 
Final_CSV_Peaks_Automated or Final_CPA_CSV_Automated notebooks, as the output of this notebook contains 
the input for these notebooks. While the two CSV_Automated notebooks can also receive manually drawn ROI 
data excel files as inputs, the Fura_ROI_Detection_Automated notebook is recommended for cases where the 
analysis of a high quantity of experiments is desired, in order to save time.

Input:
** Talk about how tif files are obtained from imaging program ? **
The input of this notebook should consist of two tif files, within a named folder, placed into the "image_stacks" sub-folder of 
the "IMAGING" folder found in "cell_tools-master". The naming of the two tif files is important, as otherwise the notebook will not
take in the tif files properly. Specifically, the two tif files must both start with the same name as the folder they are contained in,
followed by a space, and either "Ratio" for the 340/380nm ratio tif file, or "340nm" for the 340nm tif file. The notebook is case-sensitive,
and as such the capital "R" in "Ratio" must be present.
An example is as follows:
Folder name: "190521004 - HEK hR1 G2506W #30 (500nM Cell Tryp)"
340nm tif file name: "190521004 - HEK hR1 G2506W #30 (500nM Cell Tryp) 340nm.tif"
340/380nm ratio tif file name: "190521004 - HEK hR1 G2506W #30 (500nM Cell Tryp) Ratio.tif"

There is no set limit to the number of folders that can be placed within "image_stacks" and as such there is no set limit to
the number of files the notebook is able to analyze at one time. The user should take note of their system's operating specs,
as an increase in the number of files being analyzed will also increase the runtime of the notebook.

Processing:
The notebook begins by taking in the input folders and tif files contained within them, after which these files and their names
are saved to lists. The notebook then prompts the user to select the FPS (Frames Per Second) of the tif files being analyzed.
A "Results" folder is then created, if it does not already exist, that is contained within the same "IMAGING" folder as "image_stacks".
For each folder within "image_stacks", an empty results folder is created within the "Results" folder, with the same name except for the
inclusion of "Results" at the end. Next, each 340nm tif file is normalized, blurred, and thresholded in order to create a "baseline" image
to show to the user along with a prompt to select a threshold window size. The threshold window size is the size of the window used for 
adaptive thresholding, and proper window size selection can significantly alter the success of the ROI detection.
The default threshold window size is "Small" which should be appropriate for most analyses. It is recommended to increase the window size if
the size of the cells in the baseline image are particularly large, and decrease the window size if the cells are particularly small and/or closely
spaced together. If the user is unsure of the best threshold window size, it is a good strategy to keep the "Small" threshold window size and observe
the resulting mask after the next codeblock has run. If the cells do not seem to be individually distinguishable, the "XSmall" threshold window size
should be selected. If the cells appear to have holes in them where background is visible, that are not the result of nuclei, then a larger threshold
window size should be selected.
*** These changes can be made without having to completely re-run the notebook, rather the new selection can be made and then
the subsequent codeblocks can be run after the point of user input. This is the case for all user inputs. ***
After the user has selected a threshold window size for each file being analyzed, the resulting mask is displayed alongside the original
baseline image. This is to allow the user to quickly confirm the accuracy of the thresholding. It is important to note that these are not
the finalized masks, as more processing will be occuring, however it is still useful to confirm that the initial mask looks relatively accurate. 
Alongside this dual display, the user is prompted for several inputs. These include the number of erosions, the max size of ROIs, the min size of ROIs, 
and an option to discard edge ROIs. These input values are specific to each file, so different files can have different input values. 
The number of erosions input value is to select the number of erosions the notebook will perform on the mask prior to performing 
more processes, and is useful in the rare situation where the cells are still indistinguishable even with an "XSmall" 
threshold window size. It is otherwise not recommended to change the number of erosions value from the default "0" value. The max size of ROIs 
input value is for setting the maximum size of ROIs allowed before the ROI is discarded, it is recommended to keep this input to the default
"6000" value. The min size of ROIs is similarily self-explanatory, as it is for setting the minimum size of ROIs allowed before the ROI is discarded,
except in cases where cells are particularly small, it is recommended to keep this input to the default "200" value. The option for discarding edge ROIs
is for selecting whether to discard ROIs that lie along the border or edge of the image frame. When checked, the edge ROIs are not discarded, which is the
default value.
The notebook then performs any erosions that were input by the user, followed by removing ROIs too large or too small, according to the input values
selected by the user. The notebook then performs a segmentation algorithm that utilizes watershed segmentation along with other various computer 
vision tools, and displays the newly segmented masks.
** Can go into more detail here about specifics of segmenting process if useful **
The notebook then performs a bitwiseAND operation, the purpose of which is to eliminate any ROIs which were created of cells in the 340nm image
that are not visually present in the 340/380nm ratio image. The resulting ROIs are then dilated to fill in the pixelated edges of the ROIs resulting
from this process. These resulting ROIs are then numerically labeled, and presented to the user along with a ROI elimination tool, which allows the user
to input the label numbers of ROIs that are desired for removal, along with a checkbox input to reset the mask to the original mask before any eliminations.
Once the ROIs for removal are selected, or the box for reseting the mask has been checked, the subsequent code block should be run by the user as a sort of 
"enter" or "submit" button, and the previous code block may be run again to view the resulting changes to the masks. Once the masks are to the user's liking,
the subsequent code block after the "enter/submit" code block may be run to finalize the masks. Once the masks are finalized the notebook uses the masks 
and labeled ROIs to calculate the fluorescense over time of the various cells within the frame, and various plots and csv/excel files are outputted to the 
results files created earlier. An optional code block can be run at the end of the notebook, which plots the individual fluorescense intensities of each 
pixel of each ROI, as well as plotting an average of these intensities on the same plot. This code block is left as optional due to the fact that while 
it is useful for visually showing the method of data collection used by the notebook, the plotting of a line for every pixel in the ROI for every ROI 
in the frame is time consuming and greatly increases the overall runtime of the notebook.

Output:
The notebook outputs results for each analyzed file into a file specific subfolder contained within the "results" folder contained within the same "IMAGING" 
folder that contains the "image_stacks" folder. One of these outputs is an excel csv file titled with the file name within "image_stacks" followed by 
"_TOTAL-stimALL.csv". This output contains the raw data collected by the notebook, with the first column containing time values, and each subsequent column 
containing corresponding fluorescense intensity values at said time step for each ROI detected. The final column contains the averaged values of the total 
ROIs. It is this excel csv file that is the input for the CSV notebooks. The format of this excel file is identical to the format of the excel file outputted 
when manual ROI creation is performed, except for the addition of an average column at the end. Another output of the notebook is a single frame tif file 
titled with the file name within "image_stacks" followed by "-roi_mask_labelled.tif". This output may be used along with the "_TOTAL-stimALL.csv" output in 
a movie maker notebook created by John Rugis to create a movie showing the plotted fluorescense data along with the original 340/380nm Ratio tif and the 
labeled image. This is a useful tool for displaying the relationship between the plotted data with the visual changes in the tif movie. Another output 
of the notebook is a series of excel csv files, one per ROI, which contains the same raw data found in the "_TOTAL-stimALL.csv" file but only for this 
specific ROI. The title of these csv excel files begins with "apical_region_" followed by the ROI number, then followed by the file name within 
"image_stacks" and ending with "-stimALL.csv". These csv excel files are not used as inputs for the subsequent analysis notebooks, and are instead outputted 
for ease of access to the data of each ROI if the user so wishes. Another output of the notebook is a series of pdf files, one per ROI, which show the 
plotted data from the corresponding csv excel files. It is important to note that along with one for every ROI, there is also an additional csv excel and 
pdf file representing the average of the ROI data that is outputted by the notebook. The final output of the notebook is a pdf file displaying a plot of 
the averaged data along with a plot of the standard deviation from this average every 10 seconds, regardless of the FPS selected previously by the user.


Final_CSV_Peaks_Automated / Final_CPA_CSV_Automated / Final_GSK_CSV_Automated Notebooks

Purpose:
The purpose of these notebooks is to quantitatively analyze csv excel files containing raw fluorescense data of ROIs either automatically or manually 
drawn on 340/380nm ratio tif images. While the three notebooks are similar, the Final_CPA_CSV_Automated notebook is made specifically to analyze data from 
CPA experiments and the Final_GSK_CSV_Automated notebook is amde specifically to analyze data from GSK experiments.
The Final_CSV_Peaks_Automated notebook is more broadly applicable, allowing for the analysis of data from many different types of 
experiments, specifically those in which the user is interested in peaks. The outputs of all three notebooks are the same, and allow for the user 
to quickly obtain and display the analysis results of the notebooks.

Input:
The input of these notebooks is the same "_TOTAL-stimALL.csv" excel file contained within the output of the Fura_ROI_Detection_Automated notebook, or 
the output of the TILLvision excel tool used on manually drawn ROIs within TILLvision. If the user wishes to use an excel file obtained from manual ROI 
drawing on TILLvision, it is important to have the same folder layout as the output of the Fura_ROI_Detection_Automated notebook. Specifically, within 
the "results" folder should be a named folder containing the excel file starting with the same name and ending with "_TOTAL-stimALL.csv".

Processing:
The notebooks begin by prompting the user to select which files within the "results" folder that the user would like to have analyzed. The CSV notebooks then 
create a "Analysis_Results" folder within each results folder within "results" for the selected files. The notebooks will send all outputs to this folder, in 
order to allow easy distinction between the outputs of these notebooks and the outputs of the Fura_ROI_Detection_Automated notebook. Once these selections have 
been made, the notebooks prompt the user to select which ROIs should be included in the analysis for each file, along with displaying a checkbox for if the excel 
file contains an average column. The notebooks default to having all ROIs selected and the checkbox checked, as this is the most common selection for when the 
input excel files have been outputted by the Fura_ROI_Detection_Automated notebook. If the user wishes to use excel files generated from manually drawn ROIs, 
the "Includes an Avg Col" checkbox should be unchecked as these excel files will not contain an average column. Along with these prompts the notebooks will 
also display two checkboxes, the first of which allows the user to choose whether to have the stimulation zones the same for all files during the analysis. 
While this checkbox is default unchecked, it is recommended that the user checks this box if all the files they wish to analyze are of the same experiment 
type, as it saves time. If left unchecked, the user will be able to set the stimulation timezones for each file individually. The second checkbox allows the 
user to select whether to have "bad" traces automatically removed. These "bad" traces are traces which have zero values in them, which can cause inaccuracies 
in the analysis. It is highly recommended to check this box and remove the "bad" traces, but the option to not do so is left in case for some reason the user 
does not wish to. Next, the notebooks will display plots showing the 340/380nm fluorescense over time of each ROI selected for each file. Along with each of 
these plots, the notebooks will prompt the user to select how many "anomalies" / "artifacts" the user would like to remove. An "anomaly" or "artifact" can be
identified as a short period where the plotted lines either increase or decrease significantly all at the same time that is not the result of an experimental 
factor. The user can choose to remove up to six of these "anomalies" if they so choose, but the default value is zero. If the "Check this box to make 
stimulation zones the same for all files" checkbox was left unchecked, a prompt for how many stimulation zones for the analysis will be displayed along 
with each plot as well. If instead this checkbox is checked by the user, then a single prompt for selecting how many stimulation zones for the analysis will
be displayed after all the plots have been displayed. If a number of anomalies to be removed has been selected for any of the plots, then the notebooks will 
then prompt the user to select the area of the plot(s) containing the anomaly(s). 
The sliders available to the user to make this selection have a range and orientation such that the user can line up the slider with the anomaly itself, 
rather than trying to guess the specific range of time values that contains the anomaly. A tip is to click and hold on either end 
of the slider, and move the mouse up so that it is on the plot, then move the mouse so that it is slightly to the left or right of the anomaly depending on 
which side of the slider was clicked. Once released, this method should result in an accurate isolation of the anomaly zone. The notebooks will then display 
the selected anomaly removal zones on the original plots, allowing the user to identify and correct any innacuracies in selection. Then the notebooks will 
display the plot with the anomaly(s) removed, which is done by plotting the slope from the opposite ends of the selected range, such that the anomaly(s) are 
replaced with a flatter section. Next the user will be prompted to select the stimulation time zones using a similar slider, either for each file 
specifically or for all the files cumulatively depending on the selection of the stimulation zones checkbox earlier. The number of stimulation time zones 
selected earlier will determine how many sliders are displayed, with each being used to select the range for that specific time zone. The notebooks will 
then display these stimulation zones on the plots just like the anomaly removal zones, similarily allowing the user to make adjustments based on their 
observations. Along with displaying the zones, the notebooks will also prompt the user via checkboxes to choose whether to use automatic and starting 
baselines for the analysis of each file's plot. If the automatic baselines checkbox is checked by the user, the baselines used by the notebook during the 
analysis will be automatically taken from a user selected length of time before the start of each stimulation zone. If left unchecked, the user will be 
prompted to manually select which zone of the plot should be considered the baseline. It is recommended to utilize the automatic baselines when there is 
more than one stimulation zone being used in the analysis, as otherwise only one region can be selected for the baseline. If the user previously selected 
to have the stimulation time zones the same for all files, there will instead be 2 singular checkboxes to select whether to use automatic and starting
baselines for the analysis of all the files collectively. Once this selection or selections have been made, the notebook will once again display the plots
with shaded stimulation time zones, along with prompts dependent on previously selected options by the user. If the same stimulation time zones option has 
been selected by the user, a single set of prompts will be displayed, otherwise a set of prompts will be displayed for each file's plot. Several of these 
prompts are the same regardless of previous selections, those being the peak sensitivity, slope calculation method, threshold standard deviations, and 
response percentage standard deviation prompts. If automatic baselines has not been selected for the file or all the files, then a slider will be displayed 
to select a baseline untimulated time zone. If automatic baselines has been selected, then a slider for selecting the length of the automatic baselines will 
be displayed. If including starting baselines has been selected, then a slider for selecting the length of the starting baseline zone will be displayed.
Otherwise nothing related to starting baselines will be displayed. The peak counting sensitivity prompt allows the user to select between various 
sensitivity levels for the peak counting algorithm, with higher sensitivty resulting in more peaks being identified and vice versa. The slope calculation 
method prompt allows for the user to select between different methods for slope calculation, including simple difference, three-neighbor numerical, 
polynomial fit, and gaussian fit. If for some reason, such as lack of data points, the selected slope calculation method cannot be used, then the simple 
difference method will automatically be used as a fallback by the notebooks, and the user will be notified of this change. The threshold standard deviations
prompt allows the user to choose how many standard deviations the latency threshold is above the mean, accounting for noise. The response standard deviations 
prompt allows the user to choose how many standard deviations above the mean should be considered the threshold for a "response" which is used in calculating
the percentage response output. Once these selections are made by the user, the notebook begins analyzing each ROI trace within each file accordingly.

Output:
The output of these notebooks is sent to the "Analysis_Results" folders within each files specific "results" folder. The notebooks first clear this folder 
of any previous outputs. Unlike the Final_CPA_CSV_Automated and Final_GSK_CSV_Automated notebooks, the Final_CSV_Peaks_Automated notebook outputs a 
"all_peaks" csv file for each ROI trace (and the average trace if included), which contains the time and value of all peaks found within that plot. 
All three of the notebooks also output pdfs of each regions analyzed plot, ready for presentation. 
All three of the notebooks output a pdf and png file of a plot showing all the ROI traces after the anomaly removal step had been completed. 
A png version has been included to allow for immediate inclusion of the plot into powerpoints and other digital media. Along with these outputs, all three 
notebooks also output a "std_average" csv which contains the data for the average and standard deviation of the selected ROI files, and a "std_average" pdf 
which is a plot of said data. The final and most important output of the notebooks is the "Total_data_by_zone" csv file outputted for each file. For all 
three notebooks, the total data by zone csv file contains the area under the curve, maximum, latency, rise time, and max slope for each region and grouped 
by stimulation zone if more than one stimulation zone is present. The set of rows correlating with the first stimulation zone, and then the next set of rows 
correlating with the second stimulation zone, and so on. For all three notebooks the unstimulated average and responding percentage is also included for 
each zone, with the unstimulated average being zone specific if there are multiple zones and automatic baselines has been selected. The responding percentage 
indicates the percentage of the region traces that went over the "response" threshold, which was adjusted by the response standard deviations option prompt.
A column containing the starting unstimulated average is also included by all three notebooks if selection of a starting baseline was made by the user. 
Along with these shared output values, the Final_CSV_Peaks_Automated's total data by zone csv file also includes a column for the total peak count per 
stimulation zone and value of the first peak per stimulation zone. In contrast the Final_CPA_CSV_Automated's and Final_GSK_CSV_Automated's total data by zone
csv files includes columns for the peak value of the maximum peak, without the baseline being subtracted. The maximum column remains unchanged between the 
three notebooks, and still contains the peak value of the maximum peak with the baseline subtracted. The Final_CPA_CSV_Automated's and 
Final_GSK_CSV_AUtomated's total data by zone csv files also includes a column containing the difference between the starting and unstimulated baseline 
values, if inclusion of a starting baseline was selected by the user.


## Can also include analysis section specific to each notebook discussing the process used to analyze the files in more detail, not nessecarily
required for an "Instruction Manual" though.