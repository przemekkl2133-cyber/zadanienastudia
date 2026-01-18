from sqlalchemy import Column, Integer, String, Numeric, ForeignKey
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import relationship

Base = declarative_base()

class Kategoria(Base):
    __tablename__ = 'kategoriee'

    id = Column(Integer, primary_key=True, autoincrement=True)
    nazwa = Column(String, nullable=False)
    opis = Column(String)

    # Relacja: jedna kategoria może mieć wiele produktów
    produkty = relationship("Produkt", back_populates="kategoria")

class Produkt(Base):
    __tablename__ = 'produkty'

    id = Column(Integer, primary_key=True, autoincrement=True)
    nazwa = Column(String, nullable=False)
    numer = Column(Integer)
    cena = Column(Numeric)
    kategoria_id = Column(Integer, ForeignKey('kategoriee.id'))

    # Relacja: produkt przypisany do jednej kategorii
    kategoria = relationship("Kategoria", back_populates="produkty")
