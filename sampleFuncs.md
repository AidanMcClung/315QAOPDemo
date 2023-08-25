This was my code for the individual photometry task in lightCounting.
Note that I was really lazy and knew the numbers, and students hopefully had to do a bit more.

`
#This was how I did mine:

area = 25
average = 300
pixels = xy = np.mgrid[10:14:1, 10:14:1].reshape(2,-1).T
#print(pixels)
fluxsum = 0
for coord in pixels:
    fluxsum += practice_image[coord]   
#print(fluxsum)
#print(np.sum(fluxsum))

totalFlux = 720000
backFlux = average*25
starFlux = totalFlux - backFlux

print(totalFlux,backFlux,starFlux)`

This is an answer for the first task in radialProfiling
`
#Answer: Students will need to create a SkyCoord version of the RA and DEC, 
#so that it's actually an RA and DEC and not just a number

#They will need the WCS object to do this, and can extract the RA and DEC however they like
#They will need to multiply the numbers by the degree unit though, that's important.

def getXYfromRow(table_row,wcs=sample_wcs):
    this_ra = table_row['RA'] * u.deg
    this_dec = table_row['DEC'] * u.deg
    
    this_sc = SkyCoord(this_ra,this_dec)
    
    this_pixCoord = skycoord_to_pixel(this_sc,wcs=wcs)
    #print(this_pixCoord)
    x = this_pixCoord[0]
    y = this_pixCoord[1]
    return x,y
`

For the saturation checking task in radialProfiling:
`#(possible) answer:
for star in nameLocTable: #"for ___ in table" will give us the rows of the table.
    #Fill out this loop to check all your stars. It's probably smart to print their names as well if you do have any bad ones.
    star_name = star['Name']
    pixcoord = getXYfromRow(star)
    thisrp = RadialProfile(sample_image,pixcoord,radii)
    thisrp.plot() #will actually stack them on the same plot; fancy!
    #plt.title(star_name)
    #plt.show()`
    
For plotting apertures, this is what you need: 
There shouldn't be much variation possible really...
The first could be actually; they need to load in and parse the values from their row.
`

def plotApertures(table_row):
    #Fill in your function to make the circles on the plot
    x,y = SkyCoord(table_row['RA']*u.deg,table_row['DEC']*u.deg).to_pixel(sample_wcs)
    radius = table_row['r']/ 3600 / sample_pix_to_deg
    radius_in = table_row['r_in']/ 3600 / sample_pix_to_deg
    radius_out = table_row['r_out']/ 3600 / sample_pix_to_deg
    #Hint: use plt.Circle, and give it x,y,a radius, a facecolor(none) and an edgecolor
    c1 = plt.Circle((x, y), radius=radius,lw=2,fill=False,edgecolor='white')
    c2 = plt.Circle((x,y), radius=radius_in,lw=2,fill=False,edgecolor='white')
    c3 = plt.Circle((x,y), radius=radius_out,lw=2,fill=False,edgecolor='white')
    #Make sure to actually plot your circles with plt.gca().add_patch()
    plt.gca().add_patch(c1)
    plt.gca().add_patch(c2)
    plt.gca().add_patch(c3)
    
    #Don't forget to zoom in as well!
    plt.gca().set_xlim(x-(2*radius_out),x+(2*radius_out))
    plt.gca().set_ylim(y-(2*radius_out),y+(2*radius_out))
    
    star_name = table_row['Name']
    return star_name
`
    
As a final bit for radialProfiling, this was my code cell that removed data:
`#FOR AIDANS T18 TEST:

#Create a list of the rows/stars you want to get rid of:
bad_rings = ['sA','sD','sE','sG','sH','sI','sJ','sL','sN','sS','sV','sX','sY','sAA','sAE','sAF','sAG','sAH','sAK']

#Read in the table you're going to remove them from
tabToMod = Table.read(dataFilePath+'apertures.csv')

#Remove them

bad_is = []
for i,row in enumerate(tabToMod):
    if bad_rings.count(row['Name']): bad_is.append(i)
tabToMod.remove_rows(bad_is)

#Rewrite the table, replacing the old one!
tabToMod.write(dataFilePath+'apertures.csv',overwrite=True)`
