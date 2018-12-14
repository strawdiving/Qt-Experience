## download文件

QNetWorkAccessManager

QNetworkRequest

QNetworkRequest

QNetworkReply

```c++
#include <QtNetwork/QNetworkRequest>
#include <QtNetwork/QNetworkAccessManager>
#include <QtNetwork/QNetworkReply>

class FirmwarePage : public QWidget {
    Q_OBJECT
public:
    explicit FirmwarePage(QWidget *parent = 0);
    ~FirmwarePage();
public slots:   
    void _downloadProgress(qint64 curr, qint64 total);
    void _downloadFinished(void);
    void _downloadError(QNetworkReply::NetworkError code);

private: 
 	QNetworkAccessManager* _networkManager; /// interfce to download files from the network
 	QNetworkReply* _networkReply;
    QString _filename;

 	void _downloadFirmware(filename);
}
```



```c++
void _downloadFirmware(QString &filename) {
	_networkManager = new QNetworkAccessManager(this);
    QUrl firmUrl;
    if(filename.startsWith("http:"))
    {
        _filename = filename;
        firmUrl.setUrl(filename);
    }
    QNetworkRequest request(firmUrl);
    _networkReply = _networkManager->get(request);
    Q_ASSERT(_networkReply);
    			  connect(_networkReply,SIGNAL(downloadProgress(qint64,qint64)),this,SLOT(_downl oadProgress(qint64,qint64)));
    connect(_networkReply,SIGNAL(finished()),this,SLOT(_downloadFinished()));
    connect(_networkReply,SIGNAL(error(QNetworkReply::NetworkError)),this, SLOT(_downloadError(QNetworkReply::NetworkError)));
}
/// handle the downloaded firmware file
void FirmwarePage::_downloadFinished(void)
{
 _networkManager->deleteLater();
 _networkManager = NULL;

 if(_networkReply->error()!=QNetworkReply::NoError)
 {
     return;
 }

 QString filename = QFileInfo(_filename).fileName();
 Q_ASSERT(filename.isEmpty());

 QSettings setting;
 QDir dir = QFileInfo(setting.fileName()).dir();
 QString downloadFilename = dir.filePath(filename);
 QFile* downloadFile = new QFile(downloadFilename);

 if(!downloadFile->exists())
 {
      if(!downloadFile->open(QIODevice::WriteOnly))
      {
          _error(QString("Could not save downloaded file to %1. Error: %2").arg(downloadFilename).arg(downloadFile->errorString()));
          return;
      }
 }
 downloadFile->write(_networkReply->readAll());
 downloadFile->close();
   }
 else{
 qDebug()<<"Unsupported file format";
 return;
 }
}

/// handle download errors
void FirmwarePage::_downloadError(QNetworkReply::NetworkError code)
{
    QString errorMsg;

    if (code == QNetworkReply::OperationCanceledError) {
        errorMsg = "Download cancelled";

    } else if (code == QNetworkReply::ContentNotFoundError) {
        errorMsg = QString("Error: File Not Found. Please check if the firmware version is available.");

    } else {
        errorMsg = QString("Error during download. Error: %1").arg(code);
    }
    _errorCancel(errorMsg);
}


```

