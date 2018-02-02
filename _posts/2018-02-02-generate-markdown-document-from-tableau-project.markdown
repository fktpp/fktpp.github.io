---
layout: post
title: Generate markdown document from tableau project
date: 2018-02-02 11:02:36 +0800
categories: tableau xml xslt documentation
---

Just survived from a multiple tableau integrated project, the project was not very well planned. So as a result, I have to prvide some tableau project detail infomation document afterwards.

Recorded every tableau projects' data source connection, table relation and delivered location setting requires someone to open the project both on tableau desktop and tableau server. As tableau TDS file is actually just a XML file, to avoid the boring part, I came out following two simple XML style sheet[^1] for document generation.

``` xsl
<xsl:stylesheet xmlns:xsl="http://www.w3.org/1999/XSL/Transform" version="2.0">

    <xsl:output method="text"/>

    <xsl:strip-space elements="*"/>

    <xsl:key name="menu" match="tableauwb" use="@name"/>

    <xsl:template match="/">
        <xsl:text># </xsl:text>
        <xsl:value-of select="substring-after(base-uri(), 'tableau/')"/>
        <xsl:text> #</xsl:text>
        <xsl:text>&#xa;</xsl:text>
        <xsl:text>&#xa;</xsl:text>
        <xsl:text>发布之后的工程名： </xsl:text>
        <xsl:text>**</xsl:text>
        <xsl:value-of select="/workbook[1]/repository-location/@id"/>
        <xsl:text>**</xsl:text>
        <xsl:text>&#xa;</xsl:text>
        <xsl:text>&#xa;</xsl:text>
        <xsl:text>菜单位置：</xsl:text>
        <xsl:text>&#xa;</xsl:text>
        <xsl:text>&#xa;</xsl:text>
        <xsl:apply-templates select="document('tableaumenus.xml')" mode="menus">
            <xsl:with-param name="menu-name" select="/workbook[1]/repository-location/@id"/>
        </xsl:apply-templates>
        <xsl:text>&#xa;</xsl:text>
        <xsl:text>&#xa;</xsl:text>
        <xsl:apply-templates mode="datasource" select="/workbook/datasources/datasource[@caption]"/>
        <xsl:apply-templates mode="worksheets" select="/workbook/worksheets"/>
    </xsl:template>

    <xsl:template mode="datasource" match="datasource">
        <xsl:text>## 数据源: </xsl:text>
        <xsl:value-of select="attribute::caption"/>
        <xsl:text> ##</xsl:text>
        <xsl:text>&#xa;</xsl:text>
        <xsl:text>&#xa;</xsl:text>
        <xsl:text>表名：</xsl:text>
        <xsl:text>&#xa;</xsl:text>
        <xsl:text>&#xa;</xsl:text>
        <xsl:apply-templates mode="table" select="connection/relation"/>
        <xsl:text>&#xa;</xsl:text>
        <xsl:if test="connection/relation[@type='join'][1]">
            <xsl:text>关联关系：</xsl:text>
            <xsl:text>&#xa;</xsl:text>
            <xsl:text>&#xa;</xsl:text>
        </xsl:if>
        <xsl:apply-templates mode="relation" select="connection//relation"/>
        <xsl:text>&#xa;</xsl:text>
    </xsl:template>

    <xsl:template mode="table" match="relation[@type='table']">
        <xsl:text> - </xsl:text>
        <xsl:value-of select="attribute::table"/>
        <xsl:text>&#xa;</xsl:text>
    </xsl:template>

    <xsl:template mode="relation" match="relation[@type='join']">
        <xsl:text> - **</xsl:text>
        <xsl:value-of select="attribute::join"/>
        <xsl:text> </xsl:text>
        <xsl:value-of select="attribute::type"/>
        <xsl:text>** </xsl:text>
        <xsl:value-of select="clause/expression/expression[1]/@op"/>
        <xsl:value-of select="clause/expression/@op"/>
        <xsl:value-of select="clause/expression/expression[2]/@op"/>
        <xsl:text>&#xa;</xsl:text>
    </xsl:template>

    <xsl:template mode="worksheets" match="worksheets">
        <xsl:text>## 工作簿 ##</xsl:text>
        <xsl:text>&#xa;</xsl:text>
        <xsl:text>&#xa;</xsl:text>
        <xsl:apply-templates mode="worksheets" select="worksheet"/>
        <xsl:text>&#xa;</xsl:text>
    </xsl:template>

    <xsl:template mode="worksheets" match="worksheet">
        <xsl:text> - </xsl:text>
        <xsl:value-of select="attribute::name"/>
        <xsl:text>&#xa;</xsl:text>
    </xsl:template>

    <xsl:template mode="menus" match="/tableaumenus">
        <xsl:param name="menu-name"/>

        <xsl:for-each select="key('menu', $menu-name)">
            <xsl:text> - </xsl:text>
            <xsl:value-of select="@menu"/>
            <xsl:text>&#xa;</xsl:text>
        </xsl:for-each>
    </xsl:template>
</xsl:stylesheet>

```

``` xml
<tableaumenus>
  <tableauwb name="DEPLOYEDNAME0" menu="MENUNAME0" />
  <!-- similar entries here -->
  <tableauwb name="DEPLOYEDNAMEn" menu="MENUNAMEnx" />
</tableaumenus>

```

The generation is processed during maven build of our web service project, pom setting as following (in the plugins section):

``` xml
			<plugin>
				<groupId>org.codehaus.mojo</groupId>
				<artifactId>xml-maven-plugin</artifactId>
				<executions>
					<execution>
						<goals>
							<goal>transform</goal>
						</goals>
					</execution>
				</executions>
				<configuration>
					<transformationSets>
						<transformationSet>
							<dir>tableau</dir>
							<includes>*.twb</includes>
							<stylesheet>tableau/project_info.xsl</stylesheet>
							<fileMappers>
								<fileMapper implementation="org.codehaus.plexus.components.io.filemappers.FileExtensionMapper">
									<targetExtension>.md</targetExtension>
								</fileMapper>
							</fileMappers>
						</transformationSet>
					</transformationSets>
				</configuration>
				<dependencies>
					<dependency>
						<groupId>net.sf.saxon</groupId>
						<artifactId>Saxon-HE</artifactId>
						<version>9.8.0-7</version>
					</dependency>
				</dependencies>
			</plugin>

```

[^1]: base-uri() call is a xslt 2.0 function, so we have to use xslt processor SAXON for xml processing.
