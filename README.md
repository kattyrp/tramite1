#encoding:utf-8
from django.db import models
from django.contrib.auth.models import User


IDENTIFICACION_CHOICES = (
    ('DNI', 'DNI'),
    ('RUC', 'RUC'),
    ('Otro', 'Otro'),
)

ESTADO_CHOICES = (
    ('Recibido', 'Recibido'),
    ('Espera', 'En espera'),
    ('Entregado', 'Entregado'),
    ('Archivado', 'Archivado'),
)

REMITENTE_CHOICES = (
    ('PersonaNatural', 'Persona Natural'),
    ('PersonaJuridica', 'Persona Juridica'),
    )

class Requisito(models.Model):
    nom_requisito = models.CharField(max_length = 100,verbose_name="Requisito")
    des_requisito = models.TextField(help_text='Redacta el requisito')
    
    def __unicode__(self):
        return u'%s' % (self.nom_requisito)

class Asunto(models.Model):
    nom_asunto = models.CharField(max_length = 100,verbose_name="Asunto")
    des_asunto = models.TextField(help_text='Redacta el asunto')
    
    def __unicode__(self):
        return u'%s' % (self.nom_asunto)

class Tipo_Documento(models.Model):
    nom_tipo_doc = models.CharField(max_length = 100,verbose_name="Tipo de Documento")
    des_tipo_doc = models.TextField(help_text='Redacta el tipo del documento')
    
    def __unicode__(self):
        return u'%s' % (self.nom_tipo_doc)

class Estado_Documento(models.Model):
    nom_estado_doc = models.CharField(max_length = 100,verbose_name="Estado del DOcumento")
    des_estado_doc = models.TextField(help_text='Redacta el estado del documento')
    
    def __unicode__(self):
        return u'%s' % (self.nom_estado_doc)

class Area(models.Model):
    nom_area = models.CharField(max_length = 100,verbose_name="Nombre")
    des_area = models.TextField(help_text='Redacta el area',verbose_name="Descripcion")

    def __unicode__(self):
        return u'%s' % (self.nom_area)

class Remitente(models.Model):
    documento = models.CharField(max_length=50,choices=IDENTIFICACION_CHOICES)
    nro_documento = models.CharField(max_length = 100, null=True, blank=True,verbose_name="N° Documento")
    mom_pernat = models.CharField(max_length = 100, null=True, blank=True,verbose_name="Remitente")
    #Informacion de persona Juridica
    dir_perjurd = models.CharField(max_length = 100, null=True, blank=True,verbose_name="Direccion")    
    
    def __unicode__(self):
        return u' %s ' % (self.mom_pernat)

class Usuario(User):
    Nombres = models.CharField(max_length=50)
    ApellidoPaterno = models.CharField("Ape. Pat.",max_length=50)
    ApellidoMaterno = models.CharField("Ape. Mat.",max_length=50)
    Email = models.EmailField("E-mail", null=True, blank=True)
    Area = models.ForeignKey(Area,verbose_name="Area")

    def __unicode__(self):
        return u'%s %s %s' % (self.ApellidoPaterno, self.ApellidoMaterno, self.Nombres)

    def save(self):
        if not self.user_ptr_id:
            self.username = replace_all(str(self.Nombres[0].encode('utf8')).lower()) + replace_all(str(self.ApellidoPaterno.encode('utf8')).lower())
            self.first_name = self.Nombres.encode('utf8')
            self.last_name = self.ApellidoPaterno.encode('utf8') + ' ' +  self.ApellidoMaterno.encode('utf8')
            self.email = self.Email
            contrasena = self.username
            self.set_password(contrasena)
            return super(Usuario, self).save()
        else:
            self.first_name = self.Nombres
            self.last_name = self.ApellidoPaterno + ' ' +  self.ApellidoMaterno
            self.email = self.Email
            return super(Usuario, self).save()

class Documento(models.Model):
    usuario = models.ForeignKey(User)
    area = models.ForeignKey(Area,verbose_name="Area Destinataria")
    usuarea = models.ForeignKey(Usuario,verbose_name="Usuario Destinataria",related_name="usuario derivado")
    remitente = models.ForeignKey(Remitente,verbose_name="Remitentes",related_name="remitentes")
    tipodoc = models.ForeignKey(Tipo_Documento,verbose_name="Tipo de Documento")
    asunto = models.ForeignKey(Asunto,verbose_name="Asunto")
    archivo = models.FileField(upload_to ='archivo',blank = True, null = True, verbose_name = "archivo")
    folios = models.CharField(max_length = 2,verbose_name="N° Folios") 
    observacion = models.TextField(help_text='Redacta las observaciones del documento')
    estado = models.CharField(max_length=10,default = "Entregado")    

    def __unicode__(self):
        return u'%s' % (self.asunto)

    def ver_seguimiento(self):
        return '<a href=/ver/%s/ target="_blank" >Detalles</a>' % self.id
    ver_seguimiento.allow_tags = True

    def ver_historial(self):
        self.documento_generado_set.filter(documento_id = self.id)
        return '<a href=/ver/historial/%s/ target="_blank" >Historial</a>' % self.id
    ver_historial.allow_tags = True


class Documento_generado(models.Model):
    usuario = models.ForeignKey(User)
    documento = models.ForeignKey(Documento,verbose_name="Documento")
    area = models.ForeignKey(Area,null = True, blank = True,  verbose_name="Area Destinataria")
    #destinatario = models.ForeignKey(Usuario,verbose_name="Usuario Destinatario",related_name="destinatario")
    estado = models.CharField(max_length=10) 
    observacion = models.TextField(help_text='Redacta las observaciones del documento')
    
    def __unicode__(self):
        return u'%s %s %s %s %s' % (self.usuario, self.documento, self.area, self.estado, self.observacion)

def replace_all(text):
    rep = {'ñ':'n','á':'a','é':'e','í':'i','ó':'o','ú':'u'}
    for i, j in rep.iteritems():
        text = text.replace(i, j)
    return text

