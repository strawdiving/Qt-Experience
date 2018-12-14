```c++
Qt += xml
#include <QtXml/QtXml>
#include <QXmlStreamWriter>

//open existed geo file, 读XML文件
  QString filename = QFileDialog::getOpenFileName(this,tr("打开文件"),"./",tr("track(*.xml)"));
   if(!filename.isNull())
   {
     //initial,get GPS information(lat,lon) of points
     QFile file(filename);
     if(!file.open(QIODevice::ReadOnly))
     {
         QMessageBox::warning(this, tr("Error"), tr("Can't open the Break Point file!"),QMessageBox::Cancel);
     }
     QDomDocument doc;
     doc.setContent(&file);
     file.close();

     QDomElement root = doc.documentElement();
     QDomNodeList list = root.childNodes();
     for(int i =0;i<list.count();i++)
     {
         QDomNode node = list.at(i);
         if(node.isElement())
         {
          QDomElement elem= node.toElement();
          QString name = elem.tagName();
         if(name == "BREK_POINT_POS")
         {
             QString lon = elem.attribute("lon");
             QString lat = elem.attribute("lat");
             QString pos = QString("Pos: %1,%2").arg(lon).arg(lat);
             ui->textBrowser->append(pos);
         }
         else if(name =="BREAK_POINT_SEQ")
         {
             QString pos = QString("seq: %1").arg(elem.text());
             ui->textBrowser->append(pos);
         }
         else if(name == "FENCE")
         {
             QDomNodeList fenceList = node.childNodes();
             for(int i =0;i<fenceList.count();i++)
             {
                QDomNode n = fenceList.at(i);
                if(n.isElement())
                {
                     QString lat = n.toElement().attribute("lat");
                     QString lon = n.toElement().attribute("lon");
                     QString pos = QString("Fence Point %1: %2,%3").arg(i+1).arg(lon).arg(lat);
                     ui->textBrowser->append(pos);
                }
             }

         }
     }
  }
}
```

```c++
 //open existed geo file
  QString filename = QFileDialog::getOpenFileName(this,tr("打开文件"),"./",tr("track(*.gpx)"));
   if(!filename.isNull())
   {
     //initial,get GPS information(lat,lon) of points
     QFile file(filename);
     if(!file.open(QIODevice::ReadOnly))
     {
         QMessageBox::warning(this, tr("Error"), tr("Can't open the Break Point file!"),QMessageBox::Cancel);
     }
     QDomDocument doc;
     doc.setContent(&file);
     file.close();

     QDomNodeList points = doc.elementsByTagName("trkpt");
     for(int i =0;i<points.count();i++)
     {
         QDomNode node = points.at(i);
         if(node.isElement())
         {
             QString lon = node.toElement().attribute("lon");
             QString lat = node.toElement().attribute("lat");
             QPointF point(lon.toDouble(),lat.toDouble());
             gps_points_list.append(point);
         }
    }
  }

   QFile file("break_point.xml");
   if(file.open(QIODevice::ReadWrite|QIODevice::Text))
   {
       file.resize(0);
       QXmlStreamWriter writer(&file);
       //QXmlStreamWriter writer(&file);
       float offset_angle = 4.1;
       float offset_dist = 0.0;
       float yaw_set = 1.0;

      writer.setAutoFormatting(true);
      writer.writeStartDocument();
      writer.writeStartElement("BREAK_POINT");
      //seq
      writer.writeTextElement("OFFSET_DIST",QString::number(offset_dist));
      writer.writeTextElement("OFFSET_ANGLE",QString::number(offset_angle));
      writer.writeTextElement("YAW_SET",QString::number(yaw_set));

      //gps_fence
      writer.writeStartElement("FENCE");
      for(int i=0;i<gps_points_list.count();i++)
      {
     QPointF p = gps_points_list.at(i);
     QString lat = QString::number(p.y(),'g',12);
     QString lon = QString::number(p.x(),'g',12);
     writer.writeStartElement("trkpt");
     writer.writeAttribute("lat",lat);
      writer.writeAttribute("lon",lon);
      writer.writeEndElement();
      }
      writer.writeEndElement();

      writer.writeEndElement();
      writer.writeEndDocument();
}
    //route_p_gps
   file.close();
```

