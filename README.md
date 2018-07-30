# UNIB-4IZ236-Football-league

[![codebeat badge](https://codebeat.co/badges/fb32e12d-6a48-452b-a24b-327918bf1aa1)](https://codebeat.co/projects/github-com-nicharnet-unib-4iz236-football-league-master)
[![Codacy Badge](https://api.codacy.com/project/badge/Grade/50efc37159ae46579add55cda74e44e7)](https://www.codacy.com/app/NicharNET/UNIB-4IZ236-Football-league?utm_source=github.com&amp;utm_medium=referral&amp;utm_content=NicharNET/UNIB-4IZ236-Football-league&amp;utm_campaign=Badge_Grade)
[![GitHub](https://img.shields.io/github/license/mashape/apistatus.svg)](https://github.com/NicharNET/UNIB-4IZ236-Football-league/blob/master/LICENSE)

XSLT transformation to both HTML and PDF applied on an XML file with football league data

### [https://nicharnet.github.io/UNIB-4IZ236-Football-league/](https://nicharnet.github.io/UNIB-4IZ236-Football-league/)

This is also my very first XSLT transformation to both HTML and PDF from 2016 and published now. The entire semestral work could be divided into 4 separated processes. 

## Input

The only input is an XML file.

### [index.xml](https://github.com/NicharNET/UNIB-4IZ236-Football-league/blob/master/index.html)

It's the input XML file with a root element `league` and two its descendants `detail` describing the league detail and `teams` which lists the detailed description of the team itself and all its players. A brief form appears below:

```XML
<league>
    <detail>
        <!--/* League details */ -->
    </detail>
    <teams>
        <team>
            <description>
                <!--/* Team description */ -->
            </description>
            <players>
                <player>
                    <!--/* Player description */ -->
                </player>
                <!--/* Another player */ -->
        </team>
        <!--/* Another team */ -->
    </teams>
</league>
```
## Validation

The only input XML file validated against both an XSD file and a Schematron file.

### [index.xsd](https://github.com/NicharNET/UNIB-4IZ236-Football-league/blob/master/index.xsd)

This XSD file is a validation file for the input XML which validates its entire structure, the allowed count and order of the particular elements and the data type and format of their values. The following list provides a selection of the most important rules (not ordered by the importance):

 - All the names, shortcuts and contacts have to remain distinct
 - Each team is allowed to recruit at least `11` and up to `20` players and each player is between `15` and `99` years, both inclusive
 - Some of them are talented and their skill has to match the enumeration: `free kicks`, `quick`, `powerful`, `technical` or `head`
 - All the contacts, licenses and ID's must match the format usually defined with a regular expression (Regex)
 - A player can play on one of 4 available positions: `goalkeeper`, `defender`, `midfielder` or `forward`
 - Their overall skill is represented by a double number between `1` and `40`, both inclusive round on the halves using:
     ```XML
     <xsd:assertion test="$value = (for $d in 1 to 40 return 0.5 * $d)"/>
     ````   
     
I have used the Venetian Blind design.

### [index.sch](https://github.com/NicharNET/UNIB-4IZ236-Football-league/blob/master/index.sch)

The Schematron validation is a minor and a supplementary validation file using XPath. Its only job is to assure that each team will have only and exactly one player market as a captain. A part of the overall score was the usage of a Schematron file and the captain validation is a job which suits Schematron a lot:

```XML
<pattern id="check">
    <rule context="//teams/team">
        <assert test="count(players/player[@status='captain']) = 1">
            <!-- Error message -->
        </assert>
    </rule>
</pattern>
```

## Transformation

The crucial part of the semestral work is the transformation into HTML and PDF output with mutually linked pages and aggregation functions. Each transformation has defined a set of functions applicable to the statistics of the players.

### [index.xsl](https://github.com/NicharNET/UNIB-4IZ236-Football-league/blob/master/index.xsl)

The first transformation produces the website consisted of mutually linked HTML pages deployed on the project's [GitHub Pages](https://nicharnet.github.io/UNIB-4IZ236-Football-league) with the a similar design but variable linked content.

All the transformations are applied on the root element `/` and defines immediatelly self as a template rendered to `index.html` with  to generated partial templates under the `div` containers:

```XML
<xsl:template match="/">
	<xsl:result-document href="index.html" format="html">
		<html>
			<head>
				<!-- /* SKIPPED: The header */ -->
			</head>
			<body>
				<div id="header-bar"/> <xsl:call-template name="header-bar"/>	
				<div id="header">       
                    <xsl:call-template name="header-index"/>                    <!-- /* Menu template */-->
				</div>	
				<div id="league">
					<xsl:apply-templates select="//en:detail"/>	                <!-- /* League summary template */-->
				</div>
				<div id="teams">
					<xsl:apply-templates select="//en:teams"/>                  <!-- /* Teams table template */-->
				</div>
				<div id="footer">
					<xsl:call-template name="footer"/>                          <!-- /* Footer template */-->
				</div>
			</body>
		</html>
	</xsl:result-document>
    
    <xsl:result-document href="best-players.html">                              <!-- /* Best players page render */-->
		<xsl:call-template name="best-players"/>
	</xsl:result-document>
	<xsl:result-document href="top-11.html">	                            	<!-- /* Top 11 page render */-->
		<xsl:call-template name="top-11"/>
	</xsl:result-document>
</xsl:template>
```

An brief example of the `en:teams` template responsible that each team have own page generated into `chunks` folder.

```XML
<xsl:template match="en:teams">
	<h2>Teams</h2>
	<table id="teams">
		<tr id="teams-label">
			<!-- /* SKIPPED: Table header labels */-->
		</tr>
		<xsl:for-each select="en:team">
    
            <!-- /* Sorted table of teams with links to the newly generated pages below */-->
			<xsl:sort select="@id"/>
			<tr id="teams">
				<td><a href="chunks/{translate(en:description/en:name, $uppercase, $lowercase)}.html"><xsl:value-of select="@id"/></a></td>
				<td><xsl:value-of select="en:description/en:short"/></td>
				<td><xsl:value-of select="en:description/en:name"/></td>
				<td><xsl:value-of select="en:description/en:trainer/en:name"/></td>
				<td><xsl:value-of select="en:calculatePower(en:players)"/></td>
			</tr>
     
            <!-- /* Generated page for each team */-->
			<xsl:result-document href="chunks/{translate(en:description/en:name, $uppercase, $lowercase)}.html" format="html">
				<html>
					<head>
						<title><xsl:value-of select="en:description/en:name"/></title>
						<link rel="stylesheet" type="text/css" href="../index.css"/>
					</head>
					<body>
						<!-- /* SKIPPED: Header and  watermark */-->	
						<div id="league">
							<xsl:apply-templates select="en:description"/>    <!-- /* Generated team description using another template */-->
						</div>
						<div id="teams">
							<xsl:apply-templates select="en:players"/>        <!-- /* Generated list of players using another template */-->
						</div>
						<div id="footer"/>
					</body>
				</html>
			</xsl:result-document>
		</xsl:for-each>
	</table>
</xsl:template>
```

### [index-fo.xsl](https://github.com/NicharNET/UNIB-4IZ236-Football-league/blob/master/index-fo.xsl)

On the similar principle works the transformation to PDF with a watermark. In the beginning, there is generated a table of contents with links to particular pages. Each team is rendered to the separated page. Last few pages have generated tables with the best players.

## Ouptut

### HTML

### PDF

## Quality check
I have integrated [Codebeat](https://codebeat.co) and [Codacy](https://www.codacy.com) cloud static analysis services to check the overall code quality out of curiosity. 

## Licence

MIT License

Copyright (c) 2018 Nikolas Charalambidis

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
